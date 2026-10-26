import React, { useEffect, useMemo, useState } from 'react';
import { createRoot } from 'react-dom/client';
import { BookOpen, Trophy, Gamepad2, GraduationCap, Volume2, RotateCcw, ChevronRight, Menu, X, Sparkles, Star, CircleHelp, Home, CheckCircle2, Brain, UserRound } from 'lucide-react';
import { LESSONS, QUESTIONS } from './content-data.js';
import './styles.css';
import { supabase, supabaseConfigured } from './supabase.js';

function App() {
  const [page, setPage] = useState('home');
  const [lessonId, setLessonId] = useState(LESSONS[0].id);
  const [position, setPosition] = useState(1);
  const [secondPosition, setSecondPosition] = useState(1);
  const [gameMode, setGameMode] = useState('single');
  const [currentPlayer, setCurrentPlayer] = useState(1);
  const [gameWinner, setGameWinner] = useState(null);
  const [dice, setDice] = useState(null);
  const [rolling, setRolling] = useState(false);
  const [activeQuestion, setActiveQuestion] = useState(null);
  const [chosen, setChosen] = useState(null);
  const [feedback, setFeedback] = useState(null);
  const [score, setScore] = useState(() => Number(localStorage.getItem('arabTanggaScore') || 0));
  const [solved, setSolved] = useState(() => JSON.parse(localStorage.getItem('arabTanggaSolved') || '[]'));
  const [player, setPlayer] = useState(() => localStorage.getItem('arabTanggaName') || '');
  const [nameDraft, setNameDraft] = useState(player);
  const [leaderboard, setLeaderboard] = useState([]);
  const [notice, setNotice] = useState('');
  const [mobileMenu, setMobileMenu] = useState(false);
  const [lessonFilter, setLessonFilter] = useState('Semua');
  const currentLesson = LESSONS.find(l => l.id === lessonId) || LESSONS[0];
  const vocabularyIcon = meaning => { const m=meaning.toLowerCase(); if(/buku|kitab/.test(m))return '📘'; if(/pena|pensil|penggaris|penghapus|rautan|tulis/.test(m))return '✏️'; if(/pintu/.test(m))return '🚪'; if(/jendela/.test(m))return '🪟'; if(/tas/.test(m))return '🎒'; if(/meja|kursi/.test(m))return '🪑'; if(/guru|siswa|siswi|murid/.test(m))return '🧑‍🏫'; if(/ayah|ibu|keluarga|saudara|kakek|nenek|paman|bibi|anak/.test(m))return '👨‍👩‍👧'; if(/rumah|kamar|dapur|mandi|tidur|lemari|cermin|kunci|lampu|taman|kulkas/.test(m))return '🏠'; if(/air|susu|teh|kopi|minum/.test(m))return '🥤'; if(/nasi|roti|ikan|ayam|daging|telur|apel|pisang|sayur|buah/.test(m))return '🍽️'; if(/pagi|siang|sore|malam|jam|menit|waktu|besok|kemarin/.test(m))return '⏰'; if(/pasar|harga|murah|mahal|penjual|pembeli|kilogram|wortel|kentang|tomat|bawang/.test(m))return '🛍️'; if(/satu|dua|tiga|empat|lima|enam|tujuh|delapan|sembilan|sepuluh|belas|puluh/.test(m))return '🔢'; if(/kata benda|kata kerja|partikel|predikat|subjek|objek|pelaku|rafa|nashab|jar/.test(m))return '✍️'; return '💡'; };
  const progress = Math.round((solved.length / QUESTIONS.length) * 100);
  const activePosition = currentPlayer === 1 ? position : secondPosition;
  const [activityLog, setActivityLog] = useState(() => JSON.parse(localStorage.getItem('arabTanggaActivity') || '[]'));
  const [authUser, setAuthUser] = useState(null);
  const [authProfile, setAuthProfile] = useState(null);
  const [authEmail, setAuthEmail] = useState('');
  const [authPassword, setAuthPassword] = useState('');
  const [authName, setAuthName] = useState('');
  const [authMode, setAuthMode] = useState('login');
  const [cloudProgress, setCloudProgress] = useState([]);
  const [cloudLoadedFor, setCloudLoadedFor] = useState(null);
  useEffect(() => { localStorage.setItem('arabTanggaScore', String(score)); }, [score]);
  useEffect(() => { localStorage.setItem('arabTanggaSolved', JSON.stringify(solved)); }, [solved]);
  useEffect(() => { if (player) localStorage.setItem('arabTanggaName', player); }, [player]);
  useEffect(() => { localStorage.setItem('arabTanggaActivity', JSON.stringify(activityLog)); }, [activityLog]);
  useEffect(() => { fetch('/api/leaderboard').then(r=>r.ok?r.json():[]).then(setLeaderboard).catch(()=>{}); }, []);
  useEffect(() => {
    if (!supabaseConfigured || !supabase) return;
    supabase.auth.getSession().then(({data}) => setAuthUser(data.session?.user || null));
    const {data:{subscription}} = supabase.auth.onAuthStateChange((_event, session) => setAuthUser(session?.user || null));
    return () => subscription.unsubscribe();
  }, []);
  useEffect(() => {
    if (!supabaseConfigured || !supabase || !authUser) { setAuthProfile(null); return; }
    supabase.from('profiles').select('*').eq('id', authUser.id).maybeSingle().then(({data}) => {
      setAuthProfile(data || null); if (data?.display_name) { setPlayer(data.display_name); setNameDraft(data.display_name); }
    });
  }, [authUser]);
  useEffect(() => {
    let cancelled = false;
    if (!supabaseConfigured || !supabase || !authUser) { setCloudLoadedFor(null); return; }
    setCloudLoadedFor(null);
    supabase.from('learning_progress').select('*').eq('user_id', authUser.id).then(({data,error}) => {
      if (cancelled) return;
      if (!error && data?.length) {
        const remoteIds = [...new Set(data.flatMap(row => Array.isArray(row.answered_ids) ? row.answered_ids : []))];
        setSolved(remoteIds);
        setScore(data.reduce((total,row)=>total+(Number(row.score)||0),0));
      }
      setCloudLoadedFor(authUser.id);
    });
    return () => { cancelled = true; };
  }, [authUser]);
  useEffect(() => {
    if (!supabaseConfigured || !supabase || !authUser || cloudLoadedFor !== authUser.id) return;
    const save = async () => {
      const rows = LESSONS.map(lesson => { const ids = solved.filter(id => QUESTIONS.some(q => q.id === id && q.lessonId === lesson.id)); return {user_id:authUser.id, lesson_id:lesson.id, score:ids.length*10, correct_count:ids.length, answered_ids:ids, updated_at:new Date().toISOString()}; });
      const {error} = await supabase.from('learning_progress').upsert(rows,{onConflict:'user_id,lesson_id'});
      if (error) console.warn('Progress cloud sync:', error.message);
    };
    save();
  }, [authUser, cloudLoadedFor, solved, score]);
  useEffect(() => {
    if (!supabaseConfigured || !supabase || authProfile?.role !== 'teacher') { setCloudProgress([]); return; }
    Promise.all([supabase.from('learning_progress').select('*').order('updated_at',{ascending:false}), supabase.from('profiles').select('id,display_name,role')]).then(([progressResult, profileResult]) => {
      if (!progressResult.error && !profileResult.error) { const names = Object.fromEntries((profileResult.data||[]).map(p=>[p.id,p.display_name])); setCloudProgress((progressResult.data||[]).map(row=>({...row,display_name:names[row.user_id]||'Siswa'}))); }
    });
  }, [authProfile]);
  const nav = [{id:'home',label:'Beranda',icon:Home},{id:'learn',label:'Materi',icon:BookOpen},{id:'game',label:'Ular Tangga',icon:Gamepad2},{id:'quiz',label:'Latihan Soal',icon:Brain},{id:'ranking',label:'Papan Skor',icon:Trophy},{id:'dashboard',label:'Progres belajar',icon:GraduationCap},{id:'account',label:'Akun',icon:UserRound}];
  const go = p => {setPage(p);setMobileMenu(false);setNotice('');};
  const playQuestion = (square) => {
    const pool = QUESTIONS.filter(q => q.lessonId === (square % 3 === 0 ? 'nahwu' : currentLesson.id));
    const fallback = pool.length ? pool : QUESTIONS;
    const q = fallback[Math.floor(Math.random()*fallback.length)];
    setActiveQuestion({...q, square}); setChosen(null); setFeedback(null);
  };
  const rollDice = () => {
    if (rolling || activeQuestion) return;
    setRolling(true);
    const n = Math.floor(Math.random()*6)+1;
    setTimeout(() => {
      setDice(n); setRolling(false);
      const next = Math.min(100, activePosition+n);
      if (currentPlayer === 1) setPosition(next); else setSecondPosition(next);
      playQuestion(next);
    }, 550);
  };
  const answerQuestion = () => {
    if (chosen === null || !activeQuestion) return;
    const correct = chosen === activeQuestion.answer;
    const already = solved.includes(activeQuestion.id);
    setFeedback({correct, explanation:activeQuestion.explanation});
    if (correct && !already) { setScore(s=>s+10); setSolved(s=>[...s,activeQuestion.id]); }
    if (!correct) setNotice('Tidak apa-apa, pelajari penjelasan lalu coba lagi.');
  };
  const closeQuestion = () => {
    if (activeQuestion && !feedback?.correct && activeQuestion.square === 100) { if (currentPlayer === 1) setPosition(99); else setSecondPosition(99); }
    if (feedback?.correct && activeQuestion) {
      const sq = activeQuestion.square;
      const ladders = {3:14,8:30,21:42,28:76,50:67,71:92,80:99};
      const snakes = {17:7,54:34,62:19,64:60,87:24,93:73,95:75,98:79};
      const move = ladders[sq] || snakes[sq] || sq;
      if (currentPlayer === 1) setPosition(move); else setSecondPosition(move);
      if (ladders[sq]) setNotice(`Hebat! Tangga membawamu dari ${sq} ke ${ladders[sq]}.`);
      else if (snakes[sq]) setNotice(`Ups, ular membawamu dari ${sq} ke ${snakes[sq]}. Tetap semangat!`);
      if (move === 100) { setGameWinner(currentPlayer); setNotice(`Selamat! Pemain ${currentPlayer} mencapai kotak terakhir!`); }
      setActivityLog(log => [{name: currentPlayer===1 ? (player || 'Pemain 1') : 'Pemain 2', correct:!!feedback?.correct, question:activeQuestion.question, at:new Date().toISOString()}, ...log].slice(0,100));
    }
    setActiveQuestion(null); setChosen(null); setFeedback(null);
    if (gameMode === 'two' && !gameWinner) setCurrentPlayer(p => p===1 ? 2 : 1);
  };
  const resetGame = () => {setPosition(1);setSecondPosition(1);setCurrentPlayer(1);setGameWinner(null);setDice(null);setActiveQuestion(null);setNotice('Permainan dimulai ulang. Semangat!');};
  const submitScore = async () => {
    if (!player.trim()) {setNotice('Isi nama pemain terlebih dahulu.');return;}
    localStorage.setItem('arabTanggaName',player.trim());
    try {
      const res = await fetch('/api/leaderboard',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({name:player.trim(),score})});
      if(!res.ok) throw new Error('Belum terhubung');
      const data=await res.json(); setLeaderboard(data.scores||[]); setNotice('Skor berhasil dikirim ke papan skor!');
    } catch { setNotice('Papan skor online aktif setelah deploy di Netlify. Skor pribadimu tetap tersimpan di perangkat ini.'); }
  };
  const speak = text => { if ('speechSynthesis' in window) { speechSynthesis.cancel(); const u=new SpeechSynthesisUtterance(text.replace(/[ًٌٍَُِّْٰ]/g,'')); u.lang='ar-SA'; speechSynthesis.speak(u); } else setNotice('Browser ini belum mendukung pembaca suara.'); };
  const filteredLessons = LESSONS.filter(l => lessonFilter==='Semua' || l.level===lessonFilter);
  const boardSquares = Array.from({length:100},(_,i)=>i+1);
  const snakes = {17:7,54:34,62:19,64:60,87:24,93:73,95:75,98:79};
  const ladders = {3:14,8:30,21:42,28:76,50:67,71:92,80:99};
  return <div className="app-shell">
    <aside className={`sidebar ${mobileMenu?'open':''}`}>
      <div className="brand"><div className="brand-mark">ع</div><div><b>ArabTangga</b><small>Belajar sambil bermain</small></div><button className="mobile-close" onClick={()=>setMobileMenu(false)}><X size={18}/></button></div>
      <div className="profile-mini"><div className="avatar"><UserRound size={20}/></div><div><b>{player || 'Pemain Baru'}</b><small>Penjelajah Bahasa Arab</small></div></div>
      <div className="nav-label">MENU UTAMA</div>
      <nav>{nav.map(n=><button key={n.id} className={`nav-item ${page===n.id?'active':''}`} onClick={()=>go(n.id)}><n.icon size={19}/><span>{n.label}</span>{n.id==='learn'&&<span className="nav-count">{LESSONS.length}</span>}</button>)}</nav>
      <div className="sidebar-bottom"><div className="streak"><Sparkles size={18}/><div><b>Terus berlatih!</b><small>10 poin untuk jawaban benar</small></div></div><div className="sidebar-foot">Dibuat untuk pembelajar Bahasa Arab<br/>© 2026 ArabTangga</div></div>
    </aside>
    {mobileMenu && <div className="mobile-scrim" onClick={()=>setMobileMenu(false)}/>}
    <main className="main">
      <header className="topbar"><button className="menu-toggle" onClick={()=>setMobileMenu(true)}><Menu/></button><div className="breadcrumb">ArabTangga <ChevronRight size={15}/> <span>{nav.find(n=>n.id===page)?.label}</span></div><div className="top-actions"><div className="score-pill"><Star size={16} fill="currentColor"/><b>{score}</b><span>poin</span></div><button className="avatar top-avatar" onClick={()=>go('ranking')}><UserRound size={18}/></button></div></header>
      <div className="content">
        {notice && <div className="notice"><span>{notice}</span><button onClick={()=>setNotice('')}><X size={15}/></button></div>}
        {page==='home' && <><section className="hero"><div className="hero-copy"><div className="eyebrow"><Sparkles size={15}/> PETUALANGAN BAHASA ARAB</div><h1>Belajar Bahasa Arab<br/><span>jadi lebih seru!</span></h1><p>Jelajahi materi, jawab tantangan, dan naikkan pionmu di permainan ular tangga Bahasa Arab.</p><div className="hero-actions"><button className="btn-primary" onClick={()=>go('game')}><Gamepad2 size={18}/> Mulai bermain <ChevronRight size={17}/></button><button className="btn-secondary" onClick={()=>go('learn')}>Jelajahi materi</button></div><div className="hero-stats"><div><b>{LESSONS.length}</b><span>Topik materi</span></div><div><b>{QUESTIONS.length}</b><span>Soal latihan</span></div><div><b>{solved.length}</b><span>Soal terjawab benar</span></div></div></div><div className="hero-art"><div className="art-sun">✦</div><div className="art-card"><span className="art-arabic">مَرْحَبًا</span><span className="art-translation">Marhaban! / Halo!</span><div className="art-die">🎲</div></div><div className="floating f1">ا ب ت</div><div className="floating f2">أَهْلًا</div><div className="art-path"></div></div></section>
          <section className="section-head"><div><div className="eyebrow muted">MULAI DARI SINI</div><h2>Jalur belajarmu</h2><p>Pilih aktivitas sesuai kebutuhanmu hari ini.</p></div></section>
          <div className="feature-grid"><button className="feature-card mint" onClick={()=>go('learn')}><div className="feature-icon"><BookOpen/></div><span className="feature-tag">PELAJARI</span><h3>Materi Bahasa Arab</h3><p>Kosakata, contoh kalimat, dan penjelasan sederhana.</p><span className="feature-link">Buka materi <ChevronRight size={16}/></span></button><button className="feature-card peach" onClick={()=>go('game')}><div className="feature-icon"><Gamepad2/></div><span className="feature-tag">BERMAIN</span><h3>Ular Tangga</h3><p>Lempar dadu, jawab soal, dan capai kotak ke-100.</p><span className="feature-link">Main sekarang <ChevronRight size={16}/></span></button><button className="feature-card lavender" onClick={()=>go('quiz')}><div className="feature-icon"><GraduationCap/></div><span className="feature-tag">BERLATIH</span><h3>Kuis Pengetahuan</h3><p>Uji pemahamanmu dari berbagai topik pembelajaran.</p><span className="feature-link">Mulai latihan <ChevronRight size={16}/></span></button></div>
          <section className="section-head top-gap"><div><div className="eyebrow muted">KURIKULUM RINGKAS</div><h2>Topik populer</h2></div><button className="text-button" onClick={()=>go('learn')}>Lihat semua <ChevronRight size={16}/></button></section><div className="lesson-preview">{LESSONS.slice(0,4).map(l=><button className="lesson-mini" key={l.id} onClick={()=>{setLessonId(l.id);go('learn')}}><span className="lesson-emoji">{l.icon}</span><span><b>{l.subtitle}</b><small>{l.vocabulary.length} kosakata · {l.level}</small></span><ChevronRight size={17}/></button>)}</div>
        </>}
        {page==='learn' && <><div className="page-title"><div><div className="eyebrow muted">PERPUSTAKAAN BELAJAR</div><h1>Materi Bahasa Arab</h1><p>Pelajari kosakata, pahami pola kalimat, lalu dengarkan pelafalannya.</p></div><div className="title-icon"><BookOpen size={27}/></div></div><div className="filter-row">{['Semua','Pemula','Menengah'].map(f=><button key={f} onClick={()=>setLessonFilter(f)} className={`filter-chip ${lessonFilter===f?'selected':''}`}>{f}</button>)}</div><div className="lesson-layout"><div className="lesson-list">{filteredLessons.map(l=><button key={l.id} onClick={()=>setLessonId(l.id)} className={`lesson-select ${lessonId===l.id?'selected':''}`}><span className="lesson-emoji">{l.icon}</span><span><b>{l.subtitle}</b><small>{l.title}</small></span><ChevronRight size={17}/></button>)}</div><article className="lesson-detail"><div className="lesson-detail-top"><span className="lesson-emoji big">{currentLesson.icon}</span><span className="level-badge">{currentLesson.level}</span></div><h2>{currentLesson.title}</h2><h3>{currentLesson.subtitle}</h3><p className="lesson-explain">{currentLesson.explanation}</p><h4>Kosakata penting <span>{currentLesson.vocabulary.length} kata</span></h4><div className="vocab-grid">{currentLesson.vocabulary.map((v,i)=><div className="vocab-item" key={i}><span className="vocab-pictogram" aria-hidden="true">{vocabularyIcon(v[1])}</span><div><b className="arabic" dir="rtl">{v[0]}</b><small>{v[1]}</small></div><button aria-label="Dengarkan pelafalan" onClick={()=>speak(v[0])}><Volume2 size={17}/></button></div>)}</div><h4>Contoh penggunaan</h4><div className="examples">{currentLesson.examples.map((e,i)=><div key={i}><b className="arabic" dir="rtl">{e[0]}</b><span>{e[1]}</span><button onClick={()=>speak(e[0])}><Volume2 size={16}/></button></div>)}</div><button className="btn-primary full-btn" onClick={()=>go('quiz')}>Latihan dari semua materi <ChevronRight size={17}/></button></article></div></>}
        {page==='game' && <><div className="page-title"><div><div className="eyebrow muted">ARENA TANTANGAN</div><h1>Ular Tangga Arab</h1><p>Main sendiri atau bergantian dengan teman sambil menjawab soal Bahasa Arab.</p></div><button className="btn-secondary reset-btn" onClick={resetGame}><RotateCcw size={16}/> Mulai ulang</button></div><div className="game-summary"><div><span className="summary-icon">🎲</span><span><small>{gameMode==='two' ? 'Pemain 1 · '+(player || 'Kamu') : 'Posisi pion'}</small><b>{position} <em>/ 100</em></b></span></div>{gameMode==='two'&&<div><span className="summary-icon">🧑‍🤝‍🧑</span><span><small>Pemain 2</small><b>{secondPosition} <em>/ 100</em></b></span></div>}<div><span className="summary-icon">⭐</span><span><small>Total poin</small><b>{score}</b></span></div><div><span className="summary-icon">📖</span><span><small>Soal benar</small><b>{solved.length} <em>/ {QUESTIONS.length}</em></b></span></div></div><div className="game-layout"><div className="board-wrap"><div className="board-heading"><div><b>Jalur menuju garis akhir</b><small>Jawab soal untuk mengumpulkan poin</small></div><span className="board-legend"><i className="legend-ladder"/> Tangga <i className="legend-snake"/> Ular</span></div><div className="board">{boardSquares.map(s=>{const row=Math.floor((s-1)/10);const inRow=(s-1)%10;const num=row%2===0?row*10+(10-inRow):row*10+inRow+1;return <div key={num} className={`square ${num===position?'player-square':''} ${gameMode==='two'&&num===secondPosition?'second-player-square':''} ${num===1?'start-square':''} ${num===100?'finish-square':''} ${ladders[num]?'ladder-square':''} ${snakes[num]?'snake-square':''}`}><span>{num}</span>{ladders[num]&&<small>↗</small>}{snakes[num]&&<small>↘</small>}{num===position&&<div className="pawn">🔵</div>}{gameMode==='two'&&num===secondPosition&&<div className="pawn second-pawn">🟠</div>}</div>})}</div><div className="board-note"><span>🪜</span> Tangga naik · <span>🐍</span> Ular turun · <span>♟</span> Pionmu</div></div><aside className="game-control"><div className="control-top"><span className="control-icon"><Gamepad2/></span><select className="game-mode-select" value={gameMode} onChange={e=>{setGameMode(e.target.value);resetGame();}} aria-label="Mode permainan"><option value="single">Pemain tunggal</option><option value="two">Dua pemain</option></select></div><h3>{gameWinner ? `Pemain ${gameWinner} menang!` : gameMode==='two' ? `Giliran Pemain ${currentPlayer}` : 'Giliranmu!'}</h3><p>Lempar dadu untuk melangkah. Setiap kotak akan memberimu tantangan Bahasa Arab.</p><div className={`dice ${rolling?'rolling':''}`}>{dice||'⚄'}</div><button className="btn-primary full-btn" onClick={rollDice} disabled={rolling||!!activeQuestion||!!gameWinner||(gameMode==='single'&&position===100)}>{rolling?'Mengocok dadu...':'Lempar dadu 🎲'}</button><div className="rule-box"><b>📌 Aturan singkat</b><p>1. Lempar dadu untuk maju.</p><p>2. Jawab soal dengan benar untuk +10 poin.</p><p>3. Tangga membawamu naik, ular membuatmu turun.</p><p>4. Capai kotak 100 untuk menang!</p></div>{gameWinner&&<div className="win-banner">🏆 Pemain {gameWinner} berhasil mencapai akhir!</div>}</aside></div>
          {activeQuestion&&<div className="modal-backdrop"><div className="question-modal"><div className="modal-top"><span className="question-badge"><CircleHelp size={18}/> TANTANGAN KOTAK {activeQuestion.square}</span><button onClick={closeQuestion}><X/></button></div><div className="eyebrow muted">PILIH JAWABAN YANG TEPAT</div><h2>{activeQuestion.question}</h2><div className="answer-options">{activeQuestion.options.map((o,i)=><button key={i} disabled={!!feedback} onClick={()=>setChosen(i)} className={`answer-option ${chosen===i?'chosen':''} ${feedback&&i===activeQuestion.answer?'correct':''} ${feedback&&chosen===i&&i!==activeQuestion.answer?'incorrect':''}`}><span>{String.fromCharCode(65+i)}</span>{o}{feedback&&i===activeQuestion.answer&&<CheckCircle2 size={18}/>}</button>)}</div>{feedback&&<div className={`feedback ${feedback.correct?'good':'bad'}`}><b>{feedback.correct?'Benar sekali! +10 poin 🎉':'Belum tepat, tetap semangat!'}</b><p>{feedback.explanation}</p></div>}<div className="modal-footer">{!feedback?<button className="btn-primary full-btn" disabled={chosen===null} onClick={answerQuestion}>Periksa jawaban</button>:<button className="btn-primary full-btn" onClick={closeQuestion}>Lanjutkan <ChevronRight size={17}/></button>}</div></div></div>}
        </>}
        {page==='quiz' && <Quiz questions={QUESTIONS} lessons={LESSONS} solved={solved} setSolved={setSolved} score={score} setScore={setScore} speak={speak}/>}
        {page==='ranking' && <><div className="page-title"><div><div className="eyebrow muted">PENCAPAIANMU</div><h1>Papan Skor</h1><p>Lihat perkembangan belajarmu dan rayakan setiap kemajuan.</p></div><div className="title-icon gold"><Trophy size={27}/></div></div><div className="ranking-grid"><div className="ranking-main"><div className="ranking-banner"><Trophy size={35}/><div><h2>Terus kumpulkan poin!</h2><p>Jawaban benar bernilai 10 poin. Skor pribadi tersimpan di perangkatmu.</p></div></div><div className="ranking-stats"><div><small>Total poin</small><b>{score}</b></div><div><small>Soal benar</small><b>{solved.length}</b></div><div><small>Progres latihan</small><b>{progress}%</b></div></div><div className="progress-track"><div style={{width:`${progress}%`}}/></div><h3 className="table-heading"><Trophy size={18}/> Peringkat online</h3>{leaderboard.length?<div className="score-table">{leaderboard.map((r,i)=><div className="score-row" key={i}><span className="rank-number">{i+1}</span><span className="rank-avatar">{['🥇','🥈','🥉'][i]||'🎓'}</span><b>{r.name}</b><strong>{r.score} poin</strong></div>)}</div>:<div className="empty-state"><Trophy size={30}/><b>Belum ada skor online</b><p>Setelah website dideploy ke Netlify, kirim skormu untuk mulai mengisi papan skor.</p></div>}</div><aside className="submit-score"><div className="feature-icon gold-icon"><Star/></div><h3>Kirim pencapaian</h3><p>Masukkan nama yang ingin ditampilkan di papan skor online.</p><label>Nama pemain</label><input value={nameDraft} onChange={e=>setNameDraft(e.target.value)} maxLength={24} placeholder="Contoh: Aisyah"/><button className="btn-primary full-btn" onClick={()=>{setPlayer(nameDraft.trim());setTimeout(submitScore,0)}}>Kirim skor <ChevronRight size={17}/></button><small>Gunakan nama panggilan, bukan data pribadi sensitif.</small></aside></div></>}
        {page==='account' && <><div className="page-title"><div><div className="eyebrow muted">AKUN PEMBELAJAR</div><h1>Akun ArabTangga</h1><p>Masuk agar progres dapat disimpan lintas perangkat.</p></div><div className="title-icon"><UserRound size={27}/></div></div>{!supabaseConfigured ? <div className="dashboard-panel"><h2>Aktifkan akun online</h2><p className="muted-copy">Fitur akun membutuhkan proyek Supabase gratis. Website tetap bisa dipakai tanpa akun; progres lokal tersimpan di browser ini.</p><ol className="setup-list"><li>Buat proyek di <a href="https://supabase.com" target="_blank" rel="noreferrer">supabase.com</a>.</li><li>Jalankan SQL dari file <code>supabase/schema.sql</code> melalui SQL Editor.</li><li>Salin URL proyek dan anon key ke file <code>.env</code> berdasarkan <code>.env.example</code>.</li><li>Di Netlify, tambahkan environment variables <code>VITE_SUPABASE_URL</code> dan <code>VITE_SUPABASE_ANON_KEY</code>, lalu deploy ulang.</li></ol><p className="muted-copy">Jangan pernah menaruh service-role key di website atau repository publik.</p></div> : authUser ? <div className="dashboard-panel account-card"><div className="account-avatar">{(authProfile?.display_name||authUser.email||'A').slice(0,1).toUpperCase()}</div><h2>{authProfile?.display_name || player || 'Pengguna ArabTangga'}</h2><p>{authUser.email}</p><span className="level-badge">Peran: {authProfile?.role === 'teacher' ? 'Guru' : 'Siswa'}</span><p className="muted-copy">Progres akun tersimpan di database. {authProfile?.role==='teacher'?'Dashboard guru dapat melihat rekap siswa yang sudah login.':'Jika guru meminta, gunakan email akun sekolah sesuai petunjuk.'}</p><button className="btn-secondary" onClick={async()=>{await supabase.auth.signOut();setNotice('Kamu sudah keluar dari akun.');}}>Keluar</button></div> : <div className="auth-layout"><div className="dashboard-panel auth-panel"><div className="filter-row"><button className={`filter-chip ${authMode==='login'?'selected':''}`} onClick={()=>setAuthMode('login')}>Masuk</button><button className={`filter-chip ${authMode==='signup'?'selected':''}`} onClick={()=>setAuthMode('signup')}>Daftar siswa</button></div><h2>{authMode==='login'?'Selamat datang kembali':'Buat akun siswa'}</h2><p className="muted-copy">Gunakan email aktif. Jika konfirmasi email diaktifkan, cek kotak masuk setelah mendaftar.</p>{authMode==='signup'&&<><label>Nama tampilan</label><input className="auth-input" value={authName} onChange={e=>setAuthName(e.target.value)} maxLength={40} placeholder="Nama panggilan"/></>}<label>Email</label><input className="auth-input" type="email" value={authEmail} onChange={e=>setAuthEmail(e.target.value)} placeholder="nama@email.com"/><label>Kata sandi</label><input className="auth-input" type="password" value={authPassword} onChange={e=>setAuthPassword(e.target.value)} minLength={8} placeholder="Minimal 8 karakter"/><button className="btn-primary full-btn" onClick={async()=>{if(!authEmail.trim()||authPassword.length<8){setNotice('Isi email dan kata sandi minimal 8 karakter.');return;}const result=authMode==='signup'?await supabase.auth.signUp({email:authEmail.trim(),password:authPassword,options:{data:{display_name:authName.trim()||'Siswa'}}}):await supabase.auth.signInWithPassword({email:authEmail.trim(),password:authPassword});if(result.error){setNotice(result.error.message);}else{setNotice(authMode==='signup'?'Pendaftaran dikirim. Jika diminta, konfirmasi email terlebih dahulu.':'Berhasil masuk.');setAuthPassword('');}}}>{authMode==='login'?'Masuk':'Daftar akun'}</button><small>Daftar publik otomatis mendapat peran siswa. Peran guru hanya dapat diberikan oleh pengelola melalui database.</small></div></div>}</>}
        {page==='dashboard' && <><div className="page-title"><div><div className="eyebrow muted">RINGKASAN AKTIVITAS</div><h1>Progres belajar</h1><p>Pantau latihan di perangkat ini dan unduh rekap sederhana.</p></div><div className="title-icon"><GraduationCap size={27}/></div></div><div className="dashboard-stats"><div><small>Total poin</small><strong>{score}</strong></div><div><small>Soal benar unik</small><strong>{solved.length}</strong></div><div><small>Materi tersedia</small><strong>{LESSONS.length}</strong></div><div><small>Progres bank soal</small><strong>{progress}%</strong></div></div><div className="dashboard-panel"><div className="section-head"><div><h2>Rekap per materi</h2><p>Hitungan soal benar yang tersimpan pada browser ini.</p></div><button className="btn-secondary" onClick={()=>{const rows=[['Nama pemain','Materi','Soal benar','Total soal'],...LESSONS.map(l=>[player||'Pemain',l.subtitle,solved.filter(id=>QUESTIONS.some(q=>q.id===id&&q.lessonId===l.id)).length,QUESTIONS.filter(q=>q.lessonId===l.id).length])];const csv=rows.map(r=>r.map(v=>'"'+String(v).replaceAll('"','""')+'"').join(',')).join('\n');const url=URL.createObjectURL(new Blob([csv],{type:'text/csv;charset=utf-8;'}));const a=document.createElement('a');a.href=url;a.download='rekap-arabtangga.csv';a.click();URL.revokeObjectURL(url);}}>Unduh CSV</button></div><div className="dashboard-lesson-list">{LESSONS.map(l=><div className="dashboard-lesson" key={l.id}><span>{l.icon}</span><div><b>{l.subtitle}</b><small>{l.vocabulary.length} kosakata · {QUESTIONS.filter(q=>q.lessonId===l.id).length} soal</small></div><button className="text-button" onClick={()=>{setLessonId(l.id);go('learn')}}>Buka materi</button></div>)}</div><div className="notice dashboard-warning">Mode dashboard saat ini menyimpan progres secara lokal di browser ini. Untuk memantau banyak siswa dari perangkat berbeda, aktifkan autentikasi dan database Supabase mengikuti panduan di folder <code>supabase</code>.</div></div>{authProfile?.role==='teacher'&&<div className="dashboard-panel"><h2>Rekap siswa online</h2><p className="muted-copy">Data lintas perangkat dari Supabase.</p>{cloudProgress.length?cloudProgress.map((row,i)=><div className="activity-row" key={row.id||i}><span>🎓</span><div><b>{row.display_name}</b><small>{LESSONS.find(l=>l.id===row.lesson_id)?.subtitle||row.lesson_id} · {row.correct_count} jawaban benar · {row.score} poin</small></div><time>{new Date(row.updated_at).toLocaleDateString('id-ID')}</time></div>):<p className="muted-copy">Belum ada progres siswa yang tersinkron.</p>}</div>}<div className="dashboard-panel"><h2>Aktivitas terbaru</h2>{activityLog.length?activityLog.slice(0,8).map((a,i)=><div className="activity-row" key={i}><span>{a.correct?'✅':'📘'}</span><div><b>{a.name}</b><small>{a.question}</small></div><time>{new Date(a.at).toLocaleDateString('id-ID')}</time></div>):<p className="muted-copy">Aktivitas permainan akan muncul di sini setelah kamu bermain.</p>}</div></>}
        <footer className="footer"><span>ع</span><p><b>ArabTangga</b> · Belajar Bahasa Arab dengan cara menyenangkan.</p><span className="footer-right">Dibuat untuk belajar, tumbuh, dan bermain ✨</span></footer>
      </div>
    </main>
  </div>
}
function Quiz({questions,lessons,solved,setSolved,score,setScore,speak}) {
 const [filter,setFilter]=useState('Semua'); const [index,setIndex]=useState(0); const [choice,setChoice]=useState(null); const [checked,setChecked]=useState(false); const [results,setResults]=useState({});
 const list=useMemo(()=>filter==='Semua'?questions:questions.filter(q=>q.lessonId===filter),[filter,questions]);
 useEffect(()=>{setIndex(0);setChoice(null);setChecked(false);setResults({})},[filter]);
 const q=list[index]; if(!q) return null;
 const check=()=>{if(choice===null)return;setChecked(true);const ok=choice===q.answer;setResults(r=>({...r,[q.id]:ok}));if(ok&&!solved.includes(q.id)){setScore(s=>s+10);setSolved(s=>[...s,q.id]);}};
 const next=()=>{setIndex(i=>(i+1)%list.length);setChoice(null);setChecked(false)};
 return <><div className="page-title"><div><div className="eyebrow muted">UJI PEMAHAMAN</div><h1>Latihan soal</h1><p>Kerjakan kuis dari materi yang sudah kamu pelajari.</p></div><div className="title-icon"><Brain size={27}/></div></div><div className="quiz-layout"><div className="quiz-main"><div className="quiz-top"><span>SOAL {index+1} DARI {list.length}</span><span className="quiz-score"><Star size={15}/> {score} poin</span></div><div className="progress-track"><div style={{width:`${((index+1)/list.length)*100}%`}}/></div><h2>{q.question}</h2><div className="answer-options">{q.options.map((o,i)=><button key={i} disabled={checked} onClick={()=>setChoice(i)} className={`answer-option ${choice===i?'chosen':''} ${checked&&i===q.answer?'correct':''} ${checked&&choice===i&&i!==q.answer?'incorrect':''}`}><span>{String.fromCharCode(65+i)}</span>{o}{checked&&i===q.answer&&<CheckCircle2 size={18}/>}</button>)}</div>{checked&&<div className={`feedback ${results[q.id]?'good':'bad'}`}><b>{results[q.id]?'Jawabanmu benar! +10 poin':'Belum tepat, coba pahami penjelasan ini.'}</b><p>{q.explanation}</p></div>}<div className="quiz-actions">{!checked?<button className="btn-primary" disabled={choice===null} onClick={check}>Periksa jawaban</button>:<button className="btn-primary" onClick={next}>Soal berikutnya <ChevronRight size={17}/></button>}<button className="btn-secondary" onClick={()=>speak(q.question)}><Volume2 size={17}/> Dengarkan</button></div></div><aside className="quiz-side"><h3>Pilih topik</h3><p>Latihan soal sesuai materi.</p><button className={`topic-filter ${filter==='Semua'?'selected':''}`} onClick={()=>setFilter('Semua')}><span>📚</span><b>Semua topik</b><small>{questions.length}</small></button>{lessons.map(l=><button key={l.id} className={`topic-filter ${filter===l.id?'selected':''}`} onClick={()=>setFilter(l.id)}><span>{l.icon}</span><b>{l.subtitle}</b><small>{questions.filter(q=>q.lessonId===l.id).length}</small></button>)}<div className="quiz-tip"><Sparkles size={17}/><p>Tip: baca materi terlebih dahulu. Setiap soal yang benar pertama kali akan menambah 10 poin.</p></div></aside></div></>
}
createRoot(document.getElementById('root')).render(<App/>);
