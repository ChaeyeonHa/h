import React, { useState, useEffect, useRef } from 'react';
import { Search, ChevronDown, ChevronUp, Database, FileText, Cpu, CheckCircle2, Send, User, BookOpen, Quote, Sparkles } from 'lucide-react';

export default function App() {
  // 앱 상태: 'search' | 'loading' | 'chat'
  const [appState, setAppState] = useState('search');
  
  // 입력 폼 상태
  const [formData, setFormData] = useState({
    name: '',
    affiliation: '',
    scholarUrl: ''
  });
  const [showOptional, setShowOptional] = useState(false);
  
  // 로딩 상태
  const [loadingStep, setLoadingStep] = useState(0);
  
  // 채팅 상태
  const [messages, setMessages] = useState([
    {
      role: 'ai',
      content: '안녕하세요. 제 연구나 관심 분야에 대해 궁금한 점이 있으신가요? 편하게 질문 남겨주세요.'
    }
  ]);
  const [inputMessage, setInputMessage] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const messagesEndRef = useRef(null);

  // 로딩 애니메이션 시뮬레이션
  useEffect(() => {
    if (appState === 'loading') {
      const timer1 = setTimeout(() => setLoadingStep(1), 1500);
      const timer2 = setTimeout(() => setLoadingStep(2), 3500);
      const timer3 = setTimeout(() => setLoadingStep(3), 5500);
      const timer4 = setTimeout(() => setAppState('chat'), 7000);
      
      return () => {
        clearTimeout(timer1); clearTimeout(timer2); clearTimeout(timer3); clearTimeout(timer4);
      };
    }
  }, [appState]);

  // 스크롤 이동
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const handleStartCreation = (e) => {
    e.preventDefault();
    if (!formData.name || !formData.affiliation) return;
    setAppState('loading');
  };

  // 제미나이 API 호출 함수
  const fetchGeminiResponse = async (chatHistory) => {
    const apiKey = ""; // API Key는 런타임 환경에서 자동 주입됩니다.
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
    
    // 교수님 페르소나를 강제하는 시스템 프롬프트
    const systemPrompt = `당신은 ${formData.affiliation} 소속의 ${formData.name} 교수입니다. 학생이나 동료 연구자의 질문에 실제 한국의 대학교 교수님처럼 친근하면서도 전문적이고 품위 있는 말투로 답변해 주세요. '허허', '음', '그렇군요', '제 생각에는' 등 자연스러운 추임새를 적절히 사용하세요. 이전 대화의 문맥을 잘 파악하여 답변해야 합니다.`;

    const payload = {
        contents: chatHistory,
        systemInstruction: { parts: [{ text: systemPrompt }] }
    };

    // 지수 백오프를 활용한 재시도 로직
    const delays = [1000, 2000, 4000, 8000, 16000];
    for (let i = 0; i < 6; i++) {
        try {
            const response = await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            if (!response.ok) throw new Error('API Error');
            const data = await response.json();
            return data.candidates?.[0]?.content?.parts?.[0]?.text || "답변을 생성하지 못했습니다.";
        } catch (error) {
            if (i === 5) {
                return "허허, 미안하네. 지금 연구실 서버 네트워크가 좀 불안정한 것 같군. 잠시 후에 다시 질문해주겠나?";
            }
            await new Promise(resolve => setTimeout(resolve, delays[i]));
        }
    }
  };

  const handleSendMessage = async (e) => {
    e.preventDefault();
    if (!inputMessage.trim() || isTyping) return;

    const userText = inputMessage;
    // 사용자 메시지 추가
    const newMessages = [...messages, { role: 'user', content: userText }];
    setMessages(newMessages);
    setInputMessage('');
    setIsTyping(true);

    // Gemini API 포맷으로 변환 (맥락 유지를 위해 전체 대화 내역 전송)
    const chatHistory = newMessages.map(msg => ({
        role: msg.role === 'ai' ? 'model' : 'user',
        parts: [{ text: msg.content }]
    }));

    // 제미나이 API 호출
    const aiResponseText = await fetchGeminiResponse(chatHistory);

    // AI 답변 추가
    setMessages(prev => [...prev, {
        role: 'ai',
        content: aiResponseText
    }]);
    setIsTyping(false);
  };

  // 1. 검색 UI (Search View)
  if (appState === 'search') {
    return (
      <div className="min-h-screen bg-[#050505] text-white flex flex-col items-center justify-center p-6 relative overflow-hidden font-sans">
        {/* 배경 그라데이션 효과 (이미지 참고) */}
        <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[800px] h-[800px] bg-purple-900/20 rounded-full blur-[120px] pointer-events-none"></div>
        <div className="absolute top-0 right-0 w-[500px] h-[500px] bg-blue-900/10 rounded-full blur-[100px] pointer-events-none"></div>

        <div className="z-10 w-full max-w-2xl text-center mb-12">
          <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-purple-500/10 border border-purple-500/20 text-purple-400 text-sm mb-6">
            <Sparkles size={14} />
            <span>학술 연구자를 위한 맞춤형 RAG 에이전트</span>
          </div>
          <h1 className="text-4xl md:text-5xl font-bold mb-4 tracking-tight leading-tight">
            전문가의 지식을 담은<br/>단 하나의 <span className="text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-blue-500">맞춤형 AI 에이전트</span>
          </h1>
          <p className="text-gray-400 text-lg">
            구글 스칼라 데이터를 기반으로 환각 없는 정확한 학술 질의응답을 경험하세요.
          </p>
        </div>

        <form onSubmit={handleStartCreation} className="z-10 w-full max-w-xl bg-white/5 backdrop-blur-xl border border-white/10 p-8 rounded-3xl shadow-2xl">
          <div className="space-y-5">
            <div>
              <label className="block text-sm font-medium text-gray-300 mb-1.5">교수님 / 연구자 성함 *</label>
              <div className="relative">
                <div className="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <User size={18} className="text-gray-500" />
                </div>
                <input 
                  type="text" 
                  required
                  placeholder="예: 홍길동"
                  className="w-full bg-black/50 border border-white/10 rounded-xl py-3.5 pl-11 pr-4 text-white placeholder-gray-600 focus:outline-none focus:border-purple-500 focus:ring-1 focus:ring-purple-500 transition-all"
                  value={formData.name}
                  onChange={(e) => setFormData({...formData, name: e.target.value})}
                />
              </div>
            </div>

            <div>
              <label className="block text-sm font-medium text-gray-300 mb-1.5">소속 학교 / 기관명 *</label>
              <div className="relative">
                <div className="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <BookOpen size={18} className="text-gray-500" />
                </div>
                <input 
                  type="text" 
                  required
                  placeholder="예: 한국대학교 컴퓨터공학과"
                  className="w-full bg-black/50 border border-white/10 rounded-xl py-3.5 pl-11 pr-4 text-white placeholder-gray-600 focus:outline-none focus:border-purple-500 focus:ring-1 focus:ring-purple-500 transition-all"
                  value={formData.affiliation}
                  onChange={(e) => setFormData({...formData, affiliation: e.target.value})}
                />
              </div>
            </div>

            <div className="pt-2 border-t border-white/5">
              <button 
                type="button"
                onClick={() => setShowOptional(!showOptional)}
                className="flex items-center justify-between w-full text-sm text-gray-400 hover:text-white transition-colors py-2"
              >
                <span>선택 정보 입력 (구글 스칼라 URL)</span>
                {showOptional ? <ChevronUp size={16} /> : <ChevronDown size={16} />}
              </button>
              
              {showOptional && (
                <div className="mt-3 relative animate-fadeIn">
                  <div className="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                    <Search size={18} className="text-gray-500" />
                  </div>
                  <input 
                    type="url" 
                    placeholder="https://scholar.google.com/citations?user=..."
                    className="w-full bg-black/50 border border-white/10 rounded-xl py-3 pl-11 pr-4 text-sm text-white placeholder-gray-600 focus:outline-none focus:border-purple-500 focus:ring-1 focus:ring-purple-500 transition-all"
                    value={formData.scholarUrl}
                    onChange={(e) => setFormData({...formData, scholarUrl: e.target.value})}
                  />
                  <p className="text-xs text-gray-500 mt-2 ml-1">정확한 동명이인 구분을 위해 프로필 URL을 입력하시면 좋습니다.</p>
                </div>
              )}
            </div>

            <button 
              type="submit"
              className="w-full bg-gradient-to-r from-purple-600 to-blue-600 hover:from-purple-500 hover:to-blue-500 text-white font-semibold py-4 rounded-xl shadow-[0_0_20px_rgba(147,51,234,0.3)] hover:shadow-[0_0_30px_rgba(147,51,234,0.5)] transition-all duration-300 mt-4"
            >
              AI 에이전트 생성하기
            </button>
          </div>
        </form>
      </div>
    );
  }

  // 2. 로딩 UI (Loading / Data Ingestion View)
  if (appState === 'loading') {
    const steps = [
      { icon: Search, title: "구글 스칼라 프로필 검색 중...", desc: `${formData.name} (${formData.affiliation})` },
      { icon: FileText, title: "출판물 데이터 및 오픈 액세스 PDF 추출 중...", desc: "최근 5년간 논문 우선 수집" },
      { icon: Cpu, title: "문맥 단위 텍스트 청킹 및 임베딩 변환...", desc: "text-embedding-3-small 모델 사용" },
      { icon: Database, title: "Vector DB 인덱싱 및 에이전트 초기화...", desc: "Pinecone 데이터베이스 구축 중" }
    ];

    return (
      <div className="min-h-screen bg-[#050505] text-white flex flex-col items-center justify-center p-6 relative">
        <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[600px] h-[600px] bg-blue-900/10 rounded-full blur-[100px]"></div>
        
        <div className="z-10 w-full max-w-md bg-white/5 backdrop-blur-xl border border-white/10 p-10 rounded-3xl shadow-2xl">
          <div className="flex justify-center mb-8">
            <div className="relative">
              <div className="w-16 h-16 border-4 border-purple-500/30 border-t-purple-500 rounded-full animate-spin"></div>
              <div className="absolute inset-0 flex items-center justify-center">
                <Database className="text-purple-400 w-6 h-6" />
              </div>
            </div>
          </div>
          
          <h2 className="text-2xl font-bold text-center mb-8">맞춤형 지식 DB 구축 중</h2>
          
          <div className="space-y-6">
            {steps.map((step, idx) => {
              const StepIcon = step.icon;
              const isActive = idx === loadingStep;
              const isCompleted = idx < loadingStep;
              const isPending = idx > loadingStep;

              return (
                <div key={idx} className={`flex items-start gap-4 transition-opacity duration-500 ${isPending ? 'opacity-30' : 'opacity-100'}`}>
                  <div className="mt-1">
                    {isCompleted ? (
                      <CheckCircle2 className="text-green-400 w-6 h-6" />
                    ) : isActive ? (
                      <div className="w-6 h-6 bg-purple-500/20 rounded-full flex items-center justify-center animate-pulse">
                        <div className="w-2 h-2 bg-purple-500 rounded-full"></div>
                      </div>
                    ) : (
                      <div className="w-6 h-6 rounded-full border border-gray-600 flex items-center justify-center">
                        <span className="text-xs text-gray-500">{idx + 1}</span>
                      </div>
                    )}
                  </div>
                  <div>
                    <h3 className={`font-medium ${isActive ? 'text-white' : 'text-gray-300'}`}>{step.title}</h3>
                    <p className="text-xs text-gray-500 mt-1">{step.desc}</p>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      </div>
    );
  }

  // 3. 챗봇 UI (Main Q&A View)
  return (
    <div className="min-h-screen bg-[#050505] text-white flex flex-col md:flex-row font-sans">
      {/* Sidebar: Professor Profile */}
      <div className="w-full md:w-80 bg-[#0a0a0a] border-b md:border-b-0 md:border-r border-white/10 flex flex-col">
        <div className="p-6 flex-1">
          <div className="w-20 h-20 bg-gradient-to-br from-purple-500 to-blue-600 rounded-2xl p-0.5 mb-6 shadow-lg shadow-purple-500/20">
            <div className="w-full h-full bg-[#111] rounded-[14px] flex items-center justify-center">
              <User className="text-gray-400 w-10 h-10" />
            </div>
          </div>
          <h2 className="text-2xl font-bold mb-1">{formData.name} 교수</h2>
          <p className="text-gray-400 text-sm mb-6 flex items-center gap-2">
            <BookOpen size={14} /> {formData.affiliation}
          </p>

          <div className="space-y-3">
            <div className="bg-white/5 border border-white/5 rounded-xl p-4 flex justify-between items-center">
              <span className="text-gray-400 text-sm">학습된 논문 수</span>
              <span className="font-semibold text-purple-400">84편</span>
            </div>
            <div className="bg-white/5 border border-white/5 rounded-xl p-4 flex justify-between items-center">
              <span className="text-gray-400 text-sm">누적 인용 수</span>
              <span className="font-semibold text-blue-400">3,200+</span>
            </div>
            <div className="bg-white/5 border border-white/5 rounded-xl p-4 flex justify-between items-center">
              <span className="text-gray-400 text-sm">DB 상태</span>
              <span className="flex items-center gap-1.5 text-xs text-green-400 bg-green-400/10 px-2 py-1 rounded-full">
                <div className="w-1.5 h-1.5 bg-green-400 rounded-full animate-pulse"></div>
                Active (RAG)
              </span>
            </div>
          </div>
        </div>
        
        <div className="p-6 border-t border-white/10">
          <button 
            onClick={() => setAppState('search')}
            className="w-full py-3 rounded-xl border border-white/10 text-gray-400 hover:text-white hover:bg-white/5 transition-colors text-sm"
          >
            새로운 에이전트 생성
          </button>
        </div>
      </div>

      {/* Main Chat Area */}
      <div className="flex-1 flex flex-col h-[calc(100vh-auto)] md:h-screen bg-[#080808] relative">
        <div className="absolute inset-0 bg-gradient-radial from-purple-900/10 to-transparent pointer-events-none"></div>
        
        {/* Chat Header */}
        <div className="h-16 border-b border-white/10 flex items-center px-6 bg-[#0a0a0a]/80 backdrop-blur-md z-10">
          <h3 className="font-medium text-gray-200 flex items-center gap-2">
            <Sparkles size={16} className="text-purple-500" />
            {formData.name} 교수님과의 대화
          </h3>
        </div>

        {/* Messages */}
        <div className="flex-1 overflow-y-auto p-6 space-y-6 z-10 scrollbar-hide">
          {messages.map((msg, idx) => (
            <div key={idx} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
              <div className={`max-w-[85%] md:max-w-[75%] rounded-2xl p-5 ${
                msg.role === 'user' 
                  ? 'bg-gradient-to-br from-purple-600 to-blue-700 text-white shadow-lg' 
                  : 'bg-[#151515] border border-white/10 text-gray-200 shadow-md'
              }`}>
                {msg.role === 'ai' && (
                  <div className="flex items-center gap-2 mb-3 pb-3 border-b border-white/10 text-xs text-purple-400 font-medium">
                    <User size={12} /> {formData.name} 교수
                  </div>
                )}
                
                <div className="leading-relaxed whitespace-pre-wrap text-sm md:text-base">
                  {msg.content}
                </div>
              </div>
            </div>
          ))}
          
          {/* 타이핑 인디케이터 */}
          {isTyping && (
            <div className="flex justify-start">
              <div className="max-w-[85%] md:max-w-[75%] rounded-2xl p-5 bg-[#151515] border border-white/10 text-gray-200 shadow-md">
                <div className="flex items-center gap-2 mb-3 pb-3 border-b border-white/10 text-xs text-purple-400 font-medium">
                  <User size={12} /> {formData.name} 교수
                </div>
                <div className="flex items-center gap-1.5 h-6 px-2">
                  <div className="w-2 h-2 bg-purple-500 rounded-full animate-bounce" style={{ animationDelay: '0ms' }}></div>
                  <div className="w-2 h-2 bg-purple-500 rounded-full animate-bounce" style={{ animationDelay: '150ms' }}></div>
                  <div className="w-2 h-2 bg-purple-500 rounded-full animate-bounce" style={{ animationDelay: '300ms' }}></div>
                </div>
              </div>
            </div>
          )}
          <div ref={messagesEndRef} />
        </div>

        {/* Input Area */}
        <div className="p-6 bg-[#0a0a0a] border-t border-white/10 z-10">
          <form onSubmit={handleSendMessage} className="max-w-4xl mx-auto relative flex items-end bg-[#151515] border border-white/10 rounded-2xl overflow-hidden focus-within:border-purple-500/50 focus-within:ring-1 focus-within:ring-purple-500/50 transition-all">
            <textarea
              rows="1"
              className="w-full bg-transparent text-white placeholder-gray-500 p-4 max-h-32 focus:outline-none resize-none disabled:opacity-50"
              placeholder={`교수님께 질문을 남겨보세요... (예: 교수님, 이번에 발표하신 On-device AI 논문에서...)`}
              value={inputMessage}
              onChange={(e) => {
                setInputMessage(e.target.value);
                e.target.style.height = 'auto';
                e.target.style.height = e.target.scrollHeight + 'px';
              }}
              onKeyDown={(e) => {
                if (e.key === 'Enter' && !e.shiftKey) {
                  e.preventDefault();
                  handleSendMessage(e);
                }
              }}
              disabled={isTyping}
            />
            <button 
              type="submit"
              disabled={!inputMessage.trim() || isTyping}
              className="m-2 p-3 bg-white/5 hover:bg-purple-600 text-white rounded-xl disabled:opacity-50 disabled:hover:bg-white/5 transition-colors"
            >
              <Send size={18} />
            </button>
          </form>
          <div className="text-center mt-3">
            <p className="text-[10px] text-gray-500">AI는 환각(Hallucination)을 일으킬 수 있습니다. 중요한 학술 정보는 반드시 원문을 확인하세요.</p>
          </div>
        </div>
      </div>
    </div>
  );
}
