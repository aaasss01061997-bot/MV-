import React, { useState, useEffect } from 'react';
import { 
  Music, Sparkles, Youtube, Copy, CheckCircle, Mic2, Waves, 
  RefreshCw, FileText, Share2, Search, Trash2, Download, PlayCircle
} from 'lucide-react';

// --- CONFIGURATION ---
const API_KEY = "AIzaSyA64ZzV_nc8jcne-Z6IpmB9dx8bLiMtMaQ"; // API Key ที่คุณให้มา
const MODEL_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`;

export default function LelerStudioPro() {
  const [loading, setLoading] = useState(false);
  const [ytUrl, setYtUrl] = useState('');
  const [copied, setCopied] = useState(null);
  const [data, setData] = useState({
    title: '', 
    lyrics: '', 
    sunoStyle: '', 
    vocalGuide: '', 
    arrangement: '', 
    seoTags: '', 
    imagePrompt: ''
  });

  // 1. ระบบโหลดข้อมูลเก่าจาก LocalStorage เมื่อเปิดแอป
  useEffect(() => {
    const saved = localStorage.getItem('leler_pro_project');
    if (saved) setData(JSON.parse(saved));
  }, []);

  // 2. ระบบบันทึกข้อมูลอัตโนมัติเมื่อมีการเปลี่ยนแปลง
  useEffect(() => {
    if (data.lyrics) {
      localStorage.setItem('leler_pro_project', JSON.stringify(data));
    }
  }, [data]);

  // --- LOGIC: วิเคราะห์และสร้างผลงาน ---
  const handleGenerate = async () => {
    if (!ytUrl.includes('youtube.com') && !ytUrl.includes('youtu.be')) {
      alert("กรุณาวางลิงก์ YouTube เพื่อให้ AI วิเคราะห์ครับ");
      return;
    }

    setLoading(true);
    try {
      const response = await fetch(MODEL_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: `Analyze YouTube: ${ytUrl}. Create a NEW professional song.` }] }],
          systemInstruction: { 
            parts: [{ text: `You are a Senior Music Producer. Create a JSON response:
              {
                "videoTitle": "Song Name",
                "reimaginedLyrics": "Thai lyrics with [B] breath marks and (Vocal Techniques)",
                "sunoStyle": "English prompt for Suno AI (BPM, Key, Genre)",
                "vocalGuide": "Detailed Thai instruction for singer's emotion",
                "arrangement": "Music structure details",
                "seoTags": "#hashtags for TikTok/YouTube",
                "imagePrompt": "Visual description for cover art"
              }
              Strictly Thai for lyrics/guides, English for sunoStyle.` }] 
          },
          tools: [{ "google_search": {} }],
          generationConfig: { responseMimeType: "application/json" }
        })
      });

      const resJson = await response.json();
      const result = JSON.parse(resJson.candidates[0].content.parts[0].text);
      setData({
        title: result.videoTitle,
        lyrics: result.reimaginedLyrics,
        sunoStyle: result.sunoStyle,
        vocalGuide: result.vocalGuide,
        arrangement: result.arrangement,
        seoTags: result.seoTags,
        imagePrompt: result.imagePrompt
      });
    } catch (err) {
      alert("เกิดข้อผิดพลาด: " + err.message);
    } finally {
      setLoading(false);
    }
  };

  // --- LOGIC: การจัดการไฟล์และข้อความ ---
  const copyToClipboard = (text, id) => {
    navigator.clipboard.writeText(text);
    setCopied(id);
    setTimeout(() => setCopied(null), 2000);
  };

  const downloadFile = () => {
    const content = `TITLE: ${data.title}\n\nLYRICS:\n${data.lyrics}\n\nSUNO PROMPT: ${data.sunoStyle}\n\nVOCAL GUIDE: ${data.vocalGuide}`;
    const blob = new Blob([content], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${data.title || 'leler-project'}.txt`;
    a.click();
  };

  const clearData = () => {
    if (window.confirm("ต้องการล้างข้อมูลโปรเจกต์นี้ใช่หรือไม่?")) {
      setData({ title: '', lyrics: '', sunoStyle: '', vocalGuide: '', arrangement: '', seoTags: '', imagePrompt: '' });
      localStorage.removeItem('leler_pro_project');
      setYtUrl('');
    }
  };

  return (
    <div className="min-h-screen bg-[#050505] text-slate-300 p-4 md:p-10 font-sans selection:bg-indigo-500/30">
      <div className="max-w-7xl mx-auto">
        
        {/* Navigation / Actions */}
        <div className="flex justify-between items-center mb-10">
          <div className="flex items-center gap-3">
            <div className="bg-indigo-600 p-2 rounded-xl shadow-lg shadow-indigo-500/20">
              <Waves className="text-white w-6 h-6" />
            </div>
            <h1 className="text-2xl font-black text-white tracking-tighter italic">LELER <span className="text-indigo-500">STUDIO</span></h1>
          </div>
          <div className="flex gap-2">
            <button onClick={downloadFile} className="p-2.5 bg-slate-900 border border-slate-800 rounded-xl hover:bg-slate-800 transition-all text-slate-400"><Download size={18}/></button>
            <button onClick={clearData} className="p-2.5 bg-slate-900 border border-slate-800 rounded-xl hover:bg-red-900/20 hover:border-red-900/50 transition-all text-slate-500 hover:text-red-500"><Trash2 size={18}/></button>
          </div>
        </div>

        {/* Search Engine */}
        <div className="relative max-w-3xl mx-auto mb-16">
          <div className="absolute inset-0 bg-indigo-600/20 blur-3xl -z-10 rounded-full"></div>
          <div className="bg-slate-900/80 backdrop-blur-md border border-slate-800 p-2 rounded-2xl flex flex-col md:flex-row gap-2 shadow-2xl">
            <div className="flex-grow flex items-center px-4 py-2">
              <Youtube className="text-red-500 mr-3 shrink-0" size={22} />
              <input 
                className="bg-transparent border-none w-full text-white focus:ring-0 text-sm placeholder:text-slate-600"
                placeholder="วางลิงก์ YouTube เพื่อเริ่มแกะตัวตนของเพลง..."
                value={ytUrl}
                onChange={(e) => setYtUrl(e.target.value)}
              />
            </div>
            <button 
              onClick={handleGenerate}
              disabled={loading}
              className="bg-indigo-600 hover:bg-indigo-500 text-white px-10 py-3 rounded-xl font-bold text-xs uppercase tracking-[0.2em] transition-all flex items-center justify-center gap-2 shadow-lg shadow-indigo-600/30 disabled:opacity-50"
            >
              {loading ? <RefreshCw className="animate-spin" size={16} /> : <Sparkles size={16} />}
              {loading ? "Processing" : "Generate"}
            </button>
          </div>
        </div>

        {/* Workspace */}
        <div className="grid grid-cols-1 lg:grid-cols-12 gap-8">
          
          {/* Main Content (Lyrics) */}
          <div className="lg:col-span-8 space-y-6">
            <div className="bg-[#0a0a0a] border border-slate-800 rounded-[2rem] p-8 md:p-12 shadow-2xl relative overflow-hidden">
              <div className="absolute top-0 right-0 p-8 opacity-5">
                <Music size={120} />
              </div>
              
              <div className="flex justify-between items-start mb-10 border-b border-slate-900 pb-6">
                <div>
                  <h3 className="text-indigo-500 font-bold text-[10px] uppercase tracking-[0.3em] mb-2">Vocal Script v8.5</h3>
                  <h2 className="text-3xl font-bold text-white leading-tight">{data.title || "Project Untitled"}</h2>
                </div>
                {data.lyrics && (
                  <button onClick={() => copyToClipboard(data.lyrics, 'lyrics')} className="p-3 bg-slate-900 rounded-2xl hover:bg-slate-800 transition-colors">
                    {copied === 'lyrics' ? <CheckCircle size={20} className="text-emerald-500" /> : <Copy size={20} className="text-slate-500" />}
                  </button>
                )}
              </div>

              <pre className="whitespace-pre-wrap font-serif text-xl md:text-2xl text-slate-200 leading-[2.2] italic tracking-wide">
                {data.lyrics || <span className="text-slate-800 not-italic font-sans text-sm tracking-normal">Ready for your next masterpiece...</span>}
              </pre>
            </div>
          </div>

          {/* Right Sidebar (Production Tools) */}
          <div className="lg:col-span-4 space-y-6">
            
            {/* Vocal Coach Box */}
            <div className="bg-indigo-600/5 border border-indigo-600/10 p-6 rounded-3xl group hover:border-indigo-600/30 transition-all">
              <h4 className="flex items-center gap-2 text-indigo-400 font-bold text-xs mb-4 uppercase tracking-widest">
                <Mic2 size={16}/> Vocal Delivery
              </h4>
              <p className="text-sm leading-relaxed text-slate-400 italic font-medium">
                {data.vocalGuide || "AI จะแนะนำเทคนิคการใช้โทนเสียงที่นี่..."}
              </p>
            </div>

            {/* Suno Prompt Box */}
            <div className="bg-slate-900 border border-slate-800 p-6 rounded-3xl">
              <div className="flex justify-between items-center mb-4">
                <h4 className="text-slate-500 font-bold text-[10px] uppercase tracking-widest">Suno / Udio Prompt</h4>
                <button onClick={() => copyToClipboard(data.sunoStyle, 'suno')} className="text-indigo-500 hover:text-indigo-400">
                   {copied === 'suno' ? <CheckCircle size={14} /> : <Copy size={14} />}
                </button>
              </div>
              <div className="bg-black/60 p-4 rounded-2xl font-mono text-[11px] text-indigo-300 border border-slate-800 leading-relaxed break-words">
                {data.sunoStyle || "Musical tags will appear here..."}
              </div>
            </div>

            {/* Marketing & SEO */}
            <div className="bg-emerald-500/5 border border-emerald-500/10 p-6 rounded-3xl">
              <h4 className="flex items-center gap-2 text-emerald-500 font-bold text-xs mb-4 uppercase tracking-widest">
                <Share2 size={16}/> Marketing Tags
              </h4>
              <p className="text-[11px] text-slate-500 leading-relaxed">
                {data.seoTags || "#Hashtags for social media..."}
              </p>
            </div>

            {/* Quick Tips */}
            <div className="p-6 border border-slate-900 rounded-3xl">
              <h5 className="text-slate-600 font-bold text-[9px] uppercase mb-2 flex items-center gap-2"><Info size={12}/> Pro Tip</h5>
              <p className="text-[10px] text-slate-600 italic">ใช้ [B] ในเนื้อเพลงเพื่อระบุจุดที่นักร้องควรสูดลมหายใจ เพื่อความไหลลื่นของเมโลดี้</p>
            </div>

          </div>
        </div>
      </div>
    </div>
  );
}

const Info = ({size, className}) => <Waves size={size} className={className} />;
# MV-
LIVE STAGE
