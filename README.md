<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clipz - Auto Video Clipping & Editor</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://www.youtube.com/iframe_api"></script>
    <style>
        html { scroll-behavior: smooth; }
        body { font-family: 'Inter', sans-serif; background-color: #020617; color: #f8fafc; }
        .custom-scrollbar::-webkit-scrollbar { width: 6px; height: 8px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #1e293b; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #475569; border-radius: 3px; }
        .btn-disabled { cursor: not-allowed; opacity: 0.5; }
        .loader, .spinner { border: 4px solid #384252; border-top: 4px solid #38bdf8; border-radius: 50%; animation: spin 1s linear infinite; }
        .loader { width: 48px; height: 48px; }
        .spinner { width: 16px; height: 16px; border-width: 2px; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .fade-in { animation: fadeIn 0.5s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
        #sidebar { transition: transform 0.3s ease-in-out; }
        #sidebar-overlay { transition: opacity 0.3s ease-in-out; }
        .nav-link.active { background-color: #0ea5e9; color: white; }
        .nav-link.active i { color: white !important; }
        /* Style untuk editor baru */
        .media-item-container { cursor: pointer; }
        #timeline-track { min-height: 120px; }
        .timeline-clip { position: relative; height: 100px; border: 2px solid transparent; flex-shrink: 0; cursor: grab; overflow: hidden; display: flex; background-color: #1e293b; }
        .timeline-clip-filmstrip { height: 100%; width: 100%; display: flex; position: relative; overflow: hidden; }
        .timeline-clip-frame { height: 100%; background-image: var(--bg-image); background-size: auto 100%; flex-shrink: 0; }
        .timeline-clip:active { cursor: grabbing; }
        .timeline-clip.selected { border-color: #facc15; box-shadow: 0 0 15px rgba(250, 204, 21, 0.5); }
        .timeline-clip .clip-handle { position: absolute; top: 0; bottom: 0; width: 10px; background-color: rgba(250, 204, 21, 0.7); cursor: ew-resize; z-index: 11; display: none; }
        .timeline-clip.selected .clip-handle { display: block; }
        .handle-left { left: 0; border-right: 2px solid #facc15; }
        .handle-right { right: 0; border-left: 2px solid #facc15; }
        #playhead { position: absolute; top: 0; bottom: 0; left: 0; width: 2px; background-color: #fb923c; z-index: 10; cursor: ew-resize; }
        #timeline-ruler-container { height: 24px; position: relative; cursor: pointer; }
        .ruler-tick { position: absolute; bottom: 0; width: 1px; background-color: #64748b; }
        .ruler-tick.major { height: 12px; }
        .ruler-tick.minor { height: 6px; }
        .ruler-label { position: absolute; top: 0; transform: translateX(-50%); font-size: 10px; color: #94a3b8; }
    </style>
</head>
<body class="antialiased">
    <div class="relative min-h-screen">
        <!-- Sidebar & Konten Utama -->
        <div id="sidebar-overlay" class="fixed inset-0 bg-black/60 z-30 hidden"></div>
        <aside id="sidebar" class="fixed top-0 left-0 h-full w-64 bg-slate-900 border-r border-slate-800 z-40 transform -translate-x-full">
             <div class="p-4">
                <div class="flex items-center space-x-3 mb-8">
                    <div class="bg-sky-500 p-2 rounded-lg"><i data-lucide="clapperboard" class="w-6 h-6 text-white"></i></div>
                    <h1 class="text-2xl font-bold text-white tracking-tight">Clipz</h1>
                </div>
                <nav class="flex flex-col gap-2">
                    <a href="#home" class="nav-link flex items-center gap-3 px-3 py-2 text-slate-200 rounded-md font-semibold"><i data-lucide="home" class="w-5 h-5 text-sky-400"></i><span>Highlight Download</span></a>
                    <a href="#editor" class="nav-link flex items-center gap-3 px-3 py-2 text-slate-300 hover:bg-purple-500/20 rounded-md hover:text-white"><i data-lucide="wand-2" class="w-5 h-5 text-purple-400"></i><span>Editing Manual</span></a>
                    <a href="#about" class="nav-link flex items-center gap-3 px-3 py-2 text-slate-300 hover:bg-slate-700 rounded-md hover:text-white"><i data-lucide="info" class="w-5 h-5 text-slate-400"></i><span>Tentang Kami</span></a>
                </nav>
            </div>
        </aside>

        <div id="main-content" class="flex flex-col min-h-screen">
            <header class="bg-slate-900/50 backdrop-blur-sm border-b border-slate-800 sticky top-0 z-20">
                <div class="container mx-auto px-4 sm:px-6 lg:px-8">
                    <div class="flex items-center justify-between h-16">
                        <div class="flex items-center space-x-3">
                            <button id="hamburgerBtn" class="p-2 -ml-2 text-slate-400 hover:bg-slate-800 rounded-lg hover:text-white"><i data-lucide="menu" class="w-6 h-6"></i></button>
                            <span class="bg-purple-700 text-purple-300 text-xs font-medium px-2.5 py-0.5 rounded-full">Beta 17 (Fixed)</span>
                        </div>
                    </div>
                </div>
            </header>
            
            <main id="home-page" class="page-content flex-grow container mx-auto p-4 sm:p-6 lg:p-8">
                 <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 h-full">
                    <div class="lg:col-span-2 flex flex-col gap-6">
                        <div class="bg-slate-900 rounded-2xl aspect-video flex items-center justify-center border border-slate-800 overflow-hidden">
                            <div id="videoPlaceholder" class="text-center text-slate-500"><i data-lucide="video" class="w-16 h-16 mx-auto"></i><p class="mt-2">Video player akan muncul di sini.</p></div>
                            <div id="youtubePlayer" class="w-full h-full hidden"></div>
                        </div>
                        <div class="bg-slate-900 p-4 rounded-2xl border border-slate-800 flex flex-col gap-4">
                            <div class="flex items-center gap-2">
                                <div class="relative w-full flex items-center">
                                    <i data-lucide="youtube" class="w-6 h-6 text-red-500 absolute left-3 pointer-events-none"></i>
                                    <input id="ytInput" type="text" placeholder="Tempelkan link YouTube di sini..." class="w-full pl-12 pr-4 py-2 bg-slate-800 text-white rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-sky-500 transition">
                                </div>
                                 <button id="clearUrlBtn" class="p-2 text-slate-400 bg-slate-800 hover:bg-slate-700 rounded-lg hover:text-white transition hidden flex-shrink-0" title="Hapus URL">
                                    <i data-lucide="x" class="w-5 h-5"></i>
                                </button>
                                <button id="loadVideoBtn" class="px-5 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-semibold transition whitespace-nowrap flex-shrink-0">Muat Video</button>
                            </div>
                            <div id="errorMessage" class="text-red-400 text-sm hidden"></div>
                            <div class="border-t border-slate-800 my-2"></div>
                            <div class="flex flex-wrap items-center gap-3">
                                <button id="autoClipBtn" class="flex items-center gap-2 bg-sky-500 hover:bg-sky-600 text-white font-semibold px-4 py-2 rounded-lg transition btn-disabled" disabled><i data-lucide="sparkles" class="w-4 h-4"></i> Auto Clip (Simulasi)</button>
                                <button id="downloadFullBtn" class="flex items-center gap-2 bg-emerald-500 hover:bg-emerald-600 text-white font-semibold px-4 py-2 rounded-lg transition btn-disabled" disabled><i data-lucide="video" class="w-4 h-4"></i> Download Video</button>
                                <button id="deleteAllBtn" class="flex items-center gap-2 bg-red-500/20 hover:bg-red-500/40 text-red-300 font-semibold px-4 py-2 rounded-lg transition btn-disabled" disabled><i data-lucide="trash-2" class="w-4 h-4"></i> Hapus Semua Klip</button>
                            </div>
                        </div>
                    </div>
                    <div class="bg-slate-900 rounded-2xl p-6 border border-slate-800 flex flex-col h-[85vh] lg:h-auto">
                        <h2 class="text-xl font-bold text-white">Rekomendasi Klip</h2>
                        <div id="clipListContainer" class="flex-grow overflow-y-auto custom-scrollbar -mr-3 pr-3 relative mt-4">
                            <div id="clipList" class="space-y-4"></div>
                            <div id="statusOverlay" class="absolute inset-0 flex flex-col items-center justify-center text-center text-slate-500 bg-slate-900/80 backdrop-blur-sm rounded-lg transition-opacity duration-300"></div>
                        </div>
                    </div>
                </div>
            </main>
            
            <main id="editor-page" class="page-content flex-grow p-2 sm:p-4 hidden">
                <div class="h-full flex flex-col gap-4" style="height: calc(100vh - 5rem);">
                    <div class="flex-grow grid grid-cols-1 lg:grid-cols-3 gap-4">
                        <div class="lg:col-span-1 bg-slate-900 rounded-lg border border-slate-800 flex flex-col p-4">
                            <h2 class="text-lg font-bold text-white mb-4">Media Anda</h2>
                            <div id="media-bin" class="flex-grow space-y-3 overflow-y-auto custom-scrollbar pr-2"></div>
                            <label for="media-upload" class="mt-4 w-full cursor-pointer bg-purple-600 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-lg text-center transition flex items-center justify-center gap-2"><i data-lucide="upload"></i> Impor Media</label>
                            <input type="file" id="media-upload" class="hidden" accept="video/mp4" multiple>
                        </div>
                        <div class="lg:col-span-2 bg-black rounded-lg border border-slate-800 flex flex-col items-center justify-center p-4 relative">
                             <video id="preview-player" class="max-w-full max-h-full rounded-md"></video>
                             <div id="preview-placeholder" class="text-center text-slate-600"><i data-lucide="film" class="w-16 h-16 mx-auto"></i><p class="mt-2">Pratinjau Video</p></div>
                        </div>
                    </div>
                    <div class="bg-slate-900 rounded-lg border border-slate-800 p-4 flex flex-col">
                        <div class="flex items-center justify-between mb-2">
                             <h2 class="text-lg font-bold text-white">Timeline</h2>
                             <div class="flex items-center gap-2">
                                <button id="timeline-play-pause" class="p-2 bg-slate-700 hover:bg-sky-600 text-white rounded-full transition"><i data-lucide="play" class="w-5 h-5"></i></button>
                                <button id="timeline-stop" class="p-2 bg-slate-700 hover:bg-slate-600 text-white rounded-full transition"><i data-lucide="square" class="w-5 h-5"></i></button>
                                <button id="timeline-split-btn" class="p-2 bg-slate-700 hover:bg-yellow-600 text-white rounded-full transition btn-disabled" disabled title="Potong Klip (Pilih klip terlebih dahulu)"><i data-lucide="scissors" class="w-5 h-5"></i></button>
                                <button id="timeline-delete-clip-btn" class="p-2 bg-slate-700 hover:bg-red-600 text-white rounded-full transition btn-disabled" disabled title="Hapus Klip (Pilih klip terlebih dahulu)"><i data-lucide="trash-2" class="w-5 h-5"></i></button>
                                <button id="timeline-export" class="ml-4 flex items-center gap-2 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold px-4 py-2 rounded-lg transition"><i data-lucide="download" class="w-4 h-4"></i> Ekspor Proyek</button>
                             </div>
                        </div>
                        <div id="timeline-container" class="w-full overflow-x-auto custom-scrollbar">
                             <div id="timeline-ruler-container" class="h-6 w-full relative"></div>
                             <div id="timeline-track" class="bg-slate-800/50 rounded-lg flex items-center p-2 gap-2 relative">
                                <div id="playhead"></div>
                             </div>
                        </div>
                    </div>
                </div>
            </main>
            
            <main id="about-page" class="page-content flex-grow container mx-auto p-4 sm:p-6 lg:p-8 hidden">
                <div class="max-w-4xl mx-auto">
                    <div class="text-center mb-12">
                         <div class="bg-sky-500 p-3 rounded-lg inline-block mb-4"><i data-lucide="clapperboard" class="w-10 h-10 text-white"></i></div>
                        <h1 class="text-4xl font-bold text-white">Tentang Clipz</h1>
                        <p class="mt-4 text-lg text-slate-400">Misi kami adalah menyederhanakan proses pembuatan konten video pendek untuk semua orang.</p>
                    </div>
                    <div class="bg-slate-900 border border-slate-800 rounded-2xl p-8">
                        <h2 class="text-2xl font-semibold text-white mb-6">Tim di Balik Layar</h2>
                         <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6" id="team-members"></div>
                    </div>
                </div>
            </main>
        </div>
    </div>
    
    <!-- Semua Modal -->
    <div id="confirmModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 p-4 hidden"><div class="bg-slate-800 rounded-xl shadow-lg border border-slate-700 w-full max-w-sm"><div class="p-6 text-center"><div id="modalIconContainer" class="w-12 h-12 rounded-full flex items-center justify-center mx-auto mb-4"></div><h3 id="modalTitle" class="text-lg font-semibold text-white"></h3><p id="modalMessage" class="text-sm text-slate-400 mt-2"></p></div><div class="grid grid-cols-2 gap-3 p-4 bg-slate-900/50 rounded-b-xl"><button id="confirmModalCancelBtn" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 text-white rounded-lg font-semibold transition"></button><button id="confirmModalConfirmBtn" class="px-4 py-2 rounded-lg font-semibold transition"></button></div></div></div>
    <div id="resolutionModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 p-4 hidden"><div class="bg-slate-800 rounded-xl shadow-lg border border-slate-700 w-full max-w-sm"><div class="p-6"><h3 class="text-lg font-semibold text-white text-center">Pilih Resolusi Download</h3><p class="text-sm text-slate-400 mt-2 text-center">Kualitas lebih tinggi membutuhkan waktu lebih lama.</p><div class="grid grid-cols-2 gap-4 mt-6"><button data-resolution="1080" class="resolution-btn px-4 py-2 bg-slate-700 hover:bg-sky-600 text-white rounded-lg font-semibold transition">1080p <span class="text-xs text-slate-400">(Full HD)</span></button><button data-resolution="720" class="resolution-btn px-4 py-2 bg-slate-700 hover:bg-sky-600 text-white rounded-lg font-semibold transition">720p <span class="text-xs text-slate-400">(HD)</span></button><button data-resolution="480" class="resolution-btn px-4 py-2 bg-slate-700 hover:bg-sky-600 text-white rounded-lg font-semibold transition">480p <span class="text-xs text-slate-400">(SD)</span></button><button data-resolution="360" class="resolution-btn px-4 py-2 bg-slate-700 hover:bg-sky-600 text-white rounded-lg font-semibold transition">360p <span class="text-xs text-slate-400">(Rendah)</span></button></div></div><div class="p-4 bg-slate-900/50 rounded-b-xl"><button id="resolutionModalCancelBtn" class="w-full px-4 py-2 bg-slate-600 hover:bg-slate-500 text-white rounded-lg font-semibold transition">Batal</button></div></div></div>
    <div id="cutterModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 p-4 hidden"><div class="bg-slate-800 rounded-xl shadow-lg border border-slate-700 w-full max-w-2xl"><div class="p-6"><h3 class="text-xl font-semibold text-white mb-4">Potong Klip</h3><video id="cutterPlayer" class="w-full rounded-lg bg-black mb-4" controls></video><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><div><label for="startTime" class="text-sm font-medium text-slate-400">Waktu Mulai</label><div class="flex items-center gap-2 mt-1"><input id="startTime" type="number" step="0.1" min="0" class="w-full p-2 bg-slate-700 rounded-md"><button id="setStartTimeBtn" class="px-3 py-2 bg-sky-600 hover:bg-sky-700 rounded-md text-sm font-semibold">Set</button></div></div><div><label for="endTime" class="text-sm font-medium text-slate-400">Waktu Selesai</label><div class="flex items-center gap-2 mt-1"><input id="endTime" type="number" step="0.1" min="0" class="w-full p-2 bg-slate-700 rounded-md"><button id="setEndTimeBtn" class="px-3 py-2 bg-sky-600 hover:bg-sky-700 rounded-md text-sm font-semibold">Set</button></div></div></div></div><div class="grid grid-cols-2 gap-3 p-4 bg-slate-900/50 rounded-b-xl"><button id="cutterCancelBtn" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg font-semibold">Batal</button><button id="cutterAddBtn" class="px-4 py-2 bg-purple-600 hover:bg-purple-700 rounded-lg font-semibold">Tambahkan ke Timeline</button></div></div></div>

    <script>
        // --- State ---
        let player, clipsData = [], mediaBin = [], timelineClips = [];
        let currentVideoId = null, currentVideoTitle = "", videoDuration = 0, confirmAction = null;
        let currentTimelineIndex = 0, isTimelinePlaying = false, playheadAnimationId = null, isScrubbing = false;
        let currentCutterMediaId = null, selectedClipInstanceId = null;
        let isTrimming = false, trimmingHandleSide = null;
        const PIXELS_PER_SECOND = 50; // Lebih detail
        const BACKEND_URL = 'http://localhost:3000';

        function onYouTubeIframeAPIReady() {}

        document.addEventListener('DOMContentLoaded', () => {
            lucide.createIcons();
            // --- Elemen ---
            const hamburgerBtn = document.getElementById('hamburgerBtn'), sidebar = document.getElementById('sidebar'), sidebarOverlay = document.getElementById('sidebar-overlay');
            const navLinks = document.querySelectorAll('.nav-link'), pages = document.querySelectorAll('.page-content');
            const timelineContainer = document.getElementById('timeline-container'); // Perbaikan di sini
            const timelineRulerContainer = document.getElementById('timeline-ruler-container');
            const timelineDeleteClipBtn = document.getElementById('timeline-delete-clip-btn');
            const ytInput = document.getElementById('ytInput'), loadVideoBtn = document.getElementById('loadVideoBtn'), clearUrlBtn = document.getElementById('clearUrlBtn');
            const errorMessage = document.getElementById('errorMessage'), videoPlaceholder = document.getElementById('videoPlaceholder');
            const autoClipBtn = document.getElementById('autoClipBtn'), downloadFullBtn = document.getElementById('downloadFullBtn'), deleteAllBtn = document.getElementById('deleteAllBtn');
            const clipList = document.getElementById('clipList'), statusOverlay = document.getElementById('statusOverlay');
            const confirmModal = document.getElementById('confirmModal'), modalIconContainer = document.getElementById('modalIconContainer'), modalTitle = document.getElementById('modalTitle');
            const modalMessage = document.getElementById('modalMessage'), modalConfirmBtn = document.getElementById('confirmModalConfirmBtn'), modalCancelBtn = document.getElementById('confirmModalCancelBtn');
            const resolutionModal = document.getElementById('resolutionModal'), resolutionModalCancelBtn = document.getElementById('resolutionModalCancelBtn');
            const cutterModal = document.getElementById('cutterModal'), cutterPlayer = document.getElementById('cutterPlayer'), startTimeInput = document.getElementById('startTime');
            const endTimeInput = document.getElementById('endTime'), setStartTimeBtn = document.getElementById('setStartTimeBtn'), setEndTimeBtn = document.getElementById('setEndTimeBtn');
            const cutterCancelBtn = document.getElementById('cutterCancelBtn'), cutterAddBtn = document.getElementById('cutterAddBtn');
            const mediaUpload = document.getElementById('media-upload'), mediaBinContainer = document.getElementById('media-bin'), timelineTrack = document.getElementById('timeline-track');
            const previewPlayer = document.getElementById('preview-player'), previewPlaceholder = document.getElementById('preview-placeholder'), playhead = document.getElementById('playhead');
            const timelinePlayPauseBtn = document.getElementById('timeline-play-pause'), timelineStopBtn = document.getElementById('timeline-stop'), timelineExportBtn = document.getElementById('timeline-export');
            const timelineSplitBtn = document.getElementById('timeline-split-btn');
            const teamMembersContainer = document.getElementById('team-members');

            // --- DATA ---
            const team = [ { name: 'Zen', role: 'Visi & Strategi', icon: 'brain-circuit' }, { name: 'Farrel', role: 'Teknologi & Inovasi', icon: 'cpu' }, { name: 'Alvin', role: 'Desain & UI/UX', icon: 'palette' }, { name: 'Azizah', role: 'Konten & Kreativitas', icon: 'gem' }, { name: 'Reuben', role: 'Operasional & Integrasi', icon: 'git-merge' } ];
            if (teamMembersContainer) teamMembersContainer.innerHTML = team.map(member => `<div class="bg-slate-800/50 p-4 rounded-lg border border-slate-700 flex items-center gap-4"><i data-lucide="${member.icon}" class="w-8 h-8 text-sky-400 flex-shrink-0"></i><div><h4 class="font-bold text-white">${member.name}</h4><p class="text-sm text-slate-400">${member.role}</p></div></div>`).join('');
            
            // --- FUNGSI NAVIGASI ---
            const openSidebar = () => { sidebar.classList.remove('-translate-x-full'); sidebarOverlay.classList.remove('hidden'); };
            const closeSidebar = () => { sidebar.classList.add('-translate-x-full'); sidebarOverlay.classList.add('hidden'); };
            hamburgerBtn.addEventListener('click', openSidebar);
            sidebarOverlay.addEventListener('click', closeSidebar);
            const showPage = (pageId) => {
                pages.forEach(page => page.classList.add('hidden'));
                const activePage = document.getElementById(`${pageId}-page`);
                if (activePage) activePage.classList.remove('hidden');
                navLinks.forEach(link => {
                    link.classList.remove('active', 'bg-sky-500/20');
                    if(link.getAttribute('href') === `#${pageId}`) link.classList.add('active', 'bg-sky-500/20');
                });
                closeSidebar();
            };
            navLinks.forEach(link => link.addEventListener('click', (e) => { e.preventDefault(); showPage(e.currentTarget.getAttribute('href').substring(1)); }));

            // --- FUNGSI MODAL UNIVERSAL ---
            const openConfirmModal = (config) => { 
                modalTitle.textContent = config.title;
                modalMessage.textContent = config.message;
                modalCancelBtn.textContent = config.cancelText;
                modalConfirmBtn.textContent = config.confirmText;
                modalIconContainer.innerHTML = `<i data-lucide="${config.icon}" class="w-6 h-6 ${config.iconColor}"></i>`;
                modalIconContainer.className = `w-12 h-12 rounded-full flex items-center justify-center mx-auto mb-4 ${config.iconBgColor}`;
                modalConfirmBtn.className = `px-4 py-2 rounded-lg font-semibold transition ${config.confirmBtnColor}`;
                confirmAction = config.onConfirm;
                confirmModal.classList.remove('hidden');
                lucide.createIcons();
            };
            const closeConfirmModal = () => { confirmModal.classList.add('hidden'); confirmAction = null; };
            modalConfirmBtn.addEventListener('click', () => { if (confirmAction) confirmAction(); closeConfirmModal(); });
            modalCancelBtn.addEventListener('click', closeConfirmModal);

            // --- FUNGSI HALAMAN HOME ---
            const setStatus = (type, message = '') => { 
                statusOverlay.classList.remove('hidden');
                let content = '';
                switch (type) {
                    case 'loading': content = `<div class="loader"></div><p class="mt-4 font-semibold text-sky-400">${message}</p>`; break;
                    case 'empty': content = `<i data-lucide="film" class="w-16 h-16"></i><p class="mt-4 font-semibold">Belum Ada Klip</p>`; break;
                    case 'error': content = `<i data-lucide="alert-circle" class="w-16 h-16 text-red-400"></i><p class="mt-4 font-semibold">Kesalahan</p><p class="text-sm">${message}</p>`; break;
                }
                statusOverlay.innerHTML = content;
                lucide.createIcons();
            };
            const hideStatus = () => statusOverlay.classList.add('hidden');
            const updateButtonsState = () => {
                const hasVideo = !!currentVideoId;
                const hasClips = clipsData.length > 0;
                autoClipBtn.disabled = !hasVideo;
                downloadFullBtn.disabled = !hasVideo;
                deleteAllBtn.disabled = !hasClips;
                [autoClipBtn, downloadFullBtn, deleteAllBtn].forEach(btn => btn.classList.toggle('btn-disabled', btn.disabled));
            };
            const showError = (message) => { errorMessage.textContent = message; errorMessage.classList.remove('hidden'); };
            const hideError = () => { errorMessage.textContent = ''; errorMessage.classList.add('hidden'); };
            const getYouTubeVideoId = (url) => {
                const patterns = [/(?:v=)([^&]+)/, /(?:youtu\.be\/)([^?]+)/, /(?:embed\/)([^?]+)/];
                for (const pattern of patterns) {
                    const match = url.match(pattern);
                    if (match && match[1]) return match[1];
                }
                return null;
            };
            const timeStringToSeconds = (timeStr) => {
                if (!timeStr) return 0;
                const parts = timeStr.split(':').map(Number);
                if (parts.length === 3) return parts[0] * 3600 + parts[1] * 60 + parts[2];
                if (parts.length === 2) return parts[0] * 60 + parts[1];
                return 0;
            };
            const createPlayer = (videoId) => {
                videoPlaceholder.classList.add('hidden');
                document.getElementById('youtubePlayer').classList.remove('hidden');
                if (player) player.loadVideoById(videoId);
                else player = new YT.Player('youtubePlayer', { height: '100%', width: '100%', videoId: videoId, events: { 'onReady': onPlayerReady } });
            };
            function onPlayerReady(event) {
                currentVideoTitle = event.target.getVideoData().title;
                videoDuration = event.target.getDuration();
                updateButtonsState();
            }
            loadVideoBtn.addEventListener('click', () => {
                const url = ytInput.value.trim();
                if (!url) return showError("Silakan masukkan link YouTube.");
                hideError();
                const videoId = getYouTubeVideoId(url);
                if (videoId) {
                    currentVideoId = videoId;
                    videoDuration = 0;
                    createPlayer(videoId);
                    resetClips();
                } else {
                    showError("Link YouTube tidak valid.");
                    currentVideoId = null;
                }
            });
            ytInput.addEventListener('input', () => clearUrlBtn.classList.toggle('hidden', ytInput.value.length === 0));
            const resetApplication = () => {
                ytInput.value = '';
                hideError();
                if(player && typeof player.destroy === 'function') player.destroy();
                player = null;
                document.getElementById('youtubePlayer').classList.add('hidden');
                videoPlaceholder.classList.remove('hidden');
                currentVideoId = null;
                currentVideoTitle = "";
                videoDuration = 0;
                resetClips();
                clearUrlBtn.classList.add('hidden');
            };
            clearUrlBtn.addEventListener('click', resetApplication);
            const getSimulatedAIClips = (title, duration) => {
                const keyPhrases = title.split(/ dan | vs | tentang |,|\|/i).map(s => s.trim()).filter(s => s.length > 3);
                const fallbackTitles = ["Poin Argumen Utama", "Fakta Penting", "Titik Balik Cerita", "Penjelasan Kompleks", "Statistik Kunci", "Momen Krusial"];
                const formatTime = (s) => new Date(s * 1000).toISOString().substr(11, 8);
                const generatedClips = [];
                const numClips = Math.min(Math.floor(duration / 60) + 2, 8);
                const bucketSize = duration / numClips;
                for (let i = 0; i < numClips; i++) {
                    const clipTitle = keyPhrases[i] || fallbackTitles[i % fallbackTitles.length];
                    const clipDuration = Math.floor(Math.random() * 31) + 20;
                    const start = (i * bucketSize) + (Math.random() * (bucketSize - clipDuration));
                    generatedClips.push({ title: clipTitle, start: formatTime(start), end: formatTime(start + clipDuration) });
                }
                return new Promise(resolve => setTimeout(() => resolve(generatedClips), 1500));
            };
            autoClipBtn.addEventListener('click', async () => {
                if (!currentVideoId || videoDuration <= 0) return showError("Tunggu video dimuat sepenuhnya.");
                hideError();
                resetClips();
                setStatus('loading', 'Menganalisis Video...');
                try {
                    const suggestions = await getSimulatedAIClips(currentVideoTitle, videoDuration);
                    if (suggestions && Array.isArray(suggestions)) {
                        clipsData = suggestions;
                        renderClips();
                    } else {
                        throw new Error("Gagal mendapatkan saran klip.");
                    }
                } catch (error) {
                    console.error("Simulation Error:", error);
                    setStatus('error', error.message);
                }
            });
            const renderClips = () => {
                clipList.innerHTML = clipsData.length > 0 ? clipsData.map((clip, index) => `<div class="bg-slate-800 p-3 rounded-xl flex items-start gap-4 border border-slate-700 transition hover:border-sky-500 cursor-pointer" onclick="seekTo(${timeStringToSeconds(clip.start)})"><img src="https://img.youtube.com/vi/${currentVideoId}/mqdefault.jpg" class="w-24 h-14 rounded-lg object-cover flex-shrink-0"><div class="flex-grow pt-1"><h3 class="font-bold text-white text-sm leading-tight">${clip.title}</h3><p class="text-xs text-slate-400 mt-2">Timestamp: ${clip.start} - ${clip.end}</p></div><div class="flex flex-col gap-2 pt-1"><button class="download-btn p-1 rounded-md text-slate-400 hover:bg-slate-700 hover:text-white" data-index="${index}" title="Download Klip"><i data-lucide="download-cloud" class="w-4 h-4"></i></button><button class="delete-btn p-1 rounded-md text-slate-400 hover:bg-red-500/10 hover:text-red-400" data-index="${index}" title="Hapus Klip"><i data-lucide="trash-2" class="w-4 h-4"></i></button></div></div>`).join('') : '';
                if (clipsData.length > 0) hideStatus(); else setStatus('empty');
                lucide.createIcons();
                updateButtonsState();
                document.querySelectorAll('.download-btn').forEach(btn => btn.addEventListener('click', (e) => { e.stopPropagation(); downloadClip(btn.dataset.index, btn); }));
                document.querySelectorAll('.delete-btn').forEach(btn => btn.addEventListener('click', (e) => { e.stopPropagation(); openDeleteModal(Number(btn.dataset.index)); }));
            };
            window.seekTo = (seconds) => { if(player) { player.seekTo(seconds, true); player.playVideo(); }};
            const resetClips = () => { clipsData = []; renderClips(); };
            deleteAllBtn.addEventListener('click', () => openDeleteModal(null));
            const openDeleteModal = (index) => {
                const config = { icon: 'alert-triangle', iconColor: 'text-red-400', iconBgColor: 'bg-red-500/10', cancelText: 'Batal', confirmText: 'Ya, Hapus', confirmBtnColor: 'bg-red-600 hover:bg-red-700 text-white' };
                if (index !== null && index >= 0) {
                    config.title = "Hapus Klip Ini?";
                    config.message = "Anda yakin ingin menghapus klip ini?";
                    config.onConfirm = () => { clipsData.splice(index, 1); renderClips(); };
                } else {
                    config.title = "Hapus Semua Klip?";
                    config.message = `Anda yakin ingin menghapus semua ${clipsData.length} klip?`;
                    config.onConfirm = resetClips;
                }
                openConfirmModal(config);
            };
            async function downloadClip(index, buttonElement) {
                const clip = clipsData[index];
                const originalIcon = buttonElement.innerHTML;
                buttonElement.innerHTML = '<div class="spinner"></div>';
                buttonElement.disabled = true;
                try {
                    const response = await fetch(`${BACKEND_URL}/clip`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ videoId: currentVideoId, startTime: clip.start, endTime: clip.end, title: clip.title }) });
                    if (!response.ok) throw new Error((await response.json()).error || 'Server error');
                    const blob = await response.blob();
                    const url = window.URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    const disposition = response.headers.get('Content-Disposition');
                    let filename = `[Clip]_${clip.title.replace(/[^a-zA-Z0-9]/g, '_')}.mp4`;
                    if (disposition) filename = disposition.split('filename=')[1].replace(/"/g, '');
                    a.href = url; a.download = filename; a.click();
                    URL.revokeObjectURL(url);
                } catch (e) {
                    alert(`Gagal download klip. Pastikan backend berjalan.\n\nError: ${e.message}`);
                } finally {
                    buttonElement.innerHTML = originalIcon;
                    buttonElement.disabled = false;
                    lucide.createIcons();
                }
            }
            downloadFullBtn.addEventListener('click', () => !downloadFullBtn.disabled && resolutionModal.classList.remove('hidden'));
            resolutionModalCancelBtn.addEventListener('click', () => resolutionModal.classList.add('hidden'));
            document.querySelectorAll('.resolution-btn').forEach(btn => btn.addEventListener('click', (e) => {
                resolutionModal.classList.add('hidden');
                downloadFullVideo(e.currentTarget.dataset.resolution);
            }));
            async function downloadFullVideo(resolution) {
                const originalHTML = downloadFullBtn.innerHTML;
                downloadFullBtn.innerHTML = `<div class="spinner mr-2"></div><span>Mendownload...</span>`;
                downloadFullBtn.disabled = true;
                try {
                    const response = await fetch(`${BACKEND_URL}/download-full`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ videoId: currentVideoId, title: currentVideoTitle, resolution: resolution }) });
                    if (!response.ok) throw new Error((await response.json()).error || 'Server error');
                    const blob = await response.blob();
                    const url = window.URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    const disposition = response.headers.get('Content-Disposition');
                    let filename = `${currentVideoTitle.replace(/[^a-zA-Z0-9]/g, '_')}_${resolution}p.mp4`;
                    if (disposition) filename = disposition.split('filename=')[1].replace(/"/g, '');
                    a.href = url; a.download = filename; a.click();
                    URL.revokeObjectURL(url);
                } catch (error) {
                    alert(`Gagal mengunduh video.\n\nError: ${error.message}`);
                } finally {
                    downloadFullBtn.innerHTML = `<i data-lucide="video" class="w-4 h-4"></i> Download Video`;
                    updateButtonsState();
                    lucide.createIcons();
                }
            }
            
            // --- FUNGSI EDITOR BARU (BETA 14, 15, 16, 17) ---
            const openCutter = (mediaId) => {
                const mediaItem = mediaBin.find(item => item.id === mediaId);
                if (!mediaItem) return;
                currentCutterMediaId = mediaId;
                cutterPlayer.src = mediaItem.url;
                cutterPlayer.onloadedmetadata = () => {
                    startTimeInput.value = 0;
                    endTimeInput.value = mediaItem.duration.toFixed(2);
                    startTimeInput.max = mediaItem.duration.toFixed(2);
                    endTimeInput.max = mediaItem.duration.toFixed(2);
                };
                cutterModal.classList.remove('hidden');
            };
            const closeCutter = () => {
                cutterModal.classList.add('hidden');
                cutterPlayer.pause();
                currentCutterMediaId = null;
            };
            setStartTimeBtn.addEventListener('click', () => { startTimeInput.value = cutterPlayer.currentTime.toFixed(2); });
            setEndTimeBtn.addEventListener('click', () => { endTimeInput.value = cutterPlayer.currentTime.toFixed(2); });
            cutterCancelBtn.addEventListener('click', closeCutter);
            cutterAddBtn.addEventListener('click', () => {
                const start = parseFloat(startTimeInput.value);
                const end = parseFloat(endTimeInput.value);
                const mediaItem = mediaBin.find(item => item.id === currentCutterMediaId);
                if (start >= end) return alert('Waktu Selesai harus lebih besar dari Waktu Mulai.');
                if (!mediaItem) return;
                const timelineClip = { instanceId: `clip-${Date.now()}`, ...mediaItem, startTrim: start, endTrim: end, clipDuration: end - start };
                timelineClips.push(timelineClip);
                renderTimeline();
                closeCutter();
            });
            const renderMediaBin = () => {
                mediaBinContainer.innerHTML = mediaBin.map(item => `<div class="bg-slate-800 p-2 rounded-lg flex items-center gap-3"><div class="media-item-container flex-grow flex items-center gap-3" data-id="${item.id}"><video src="${item.url}" class="w-16 h-10 rounded-md bg-black object-cover pointer-events-none"></video><span class="text-sm text-slate-300 truncate pointer-events-none">${item.name}</span></div><button class="delete-media-btn p-1 text-slate-500 hover:text-red-400" data-id="${item.id}"><i data-lucide="x-circle" class="w-4 h-4"></i></button></div>`).join('');
                addMediaBinListeners();
            };
            const addMediaBinListeners = () => {
                document.querySelectorAll('.media-item-container').forEach(item => item.addEventListener('click', (e) => openCutter(e.currentTarget.dataset.id)));
                document.querySelectorAll('.delete-media-btn').forEach(btn => btn.addEventListener('click', (e) => { e.stopPropagation(); deleteMediaItem(e.currentTarget.dataset.id); }));
                lucide.createIcons();
            };
            const deleteMediaItem = (mediaId) => {
                mediaBin = mediaBin.filter(item => item.id !== mediaId);
                timelineClips = timelineClips.filter(clip => clip.mediaId !== mediaId);
                renderMediaBin();
                renderTimeline();
            };
            const renderTimeline = () => {
                totalTimelineDuration = timelineClips.reduce((sum, clip) => sum + clip.clipDuration, 0);
                timelineTrack.innerHTML = `<div id="playhead"></div>` + timelineClips.map(clip => {
                    const filmstripHtml = Array.from({length: Math.ceil(clip.clipDuration * PIXELS_PER_SECOND / 100)}, (_, i) => 
                        `<div class="timeline-clip-frame" style="width: 100px; --bg-image: url(${clip.url}); background-position-x: -${(clip.startTrim * PIXELS_PER_SECOND) + (i * 100)}px;"></div>`
                    ).join('');
                    return `<div class="timeline-clip" data-instance-id="${clip.instanceId}" style="width: ${clip.clipDuration * PIXELS_PER_SECOND}px;" onclick="selectClip('${clip.instanceId}')">
                                <div class="timeline-clip-filmstrip">${filmstripHtml}</div>
                                <div class="handle-left clip-handle"></div><div class="handle-right clip-handle"></div>
                                <span class="absolute bottom-1 left-1 text-white text-xs truncate p-1 bg-black/50 rounded-sm">${clip.name}</span>
                            </div>`;
                }).join('');
                document.querySelectorAll('.handle-left').forEach(handle => { handle.addEventListener('mousedown', (e) => startTrimming(e, 'left')); });
                document.querySelectorAll('.handle-right').forEach(handle => { handle.addEventListener('mousedown', (e) => startTrimming(e, 'right')); });
                renderTimelineRuler();
            };
            const playTimeline = () => {
                if (timelineClips.length === 0) return;
                const playheadEl = document.getElementById('playhead');
                const playheadX = parseFloat((playheadEl.style.transform || 'translateX(0px)').replace(/[^0-9.-]/g, ''));
                const globalStartTime = playheadX / PIXELS_PER_SECOND;
                const { clip, localTime, clipIndex } = findClipAtGlobalTime(globalStartTime);
                if(!clip) return stopTimeline();
                isTimelinePlaying = true;
                timelinePlayPauseBtn.innerHTML = `<i data-lucide="pause" class="w-5 h-5"></i>`;
                lucide.createIcons();
                currentTimelineIndex = clipIndex;
                selectClip(clip.instanceId, true, true, clip.startTrim + localTime);
                updatePlayheadPosition();
            };
            const pauseTimeline = () => {
                isTimelinePlaying = false;
                previewPlayer.pause();
                timelinePlayPauseBtn.innerHTML = `<i data-lucide="play" class="w-5 h-5"></i>`;
                lucide.createIcons();
                cancelAnimationFrame(playheadAnimationId);
            };
            const stopTimeline = () => {
                pauseTimeline();
                currentTimelineIndex = 0;
                if(timelineClips.length > 0) {
                    previewPlayer.currentTime = timelineClips[0].startTrim;
                    selectClip(timelineClips[0].instanceId, false);
                } else {
                    previewPlayer.src = '';
                    previewPlayer.classList.add('hidden');
                    previewPlaceholder.classList.remove('hidden');
                }
                const playheadEl = document.getElementById('playhead');
                if(playheadEl) playheadEl.style.transform = `translateX(0px)`;
            };
            timelinePlayPauseBtn.addEventListener('click', () => isTimelinePlaying ? pauseTimeline() : playTimeline());
            timelineStopBtn.addEventListener('click', stopTimeline);
            const playClip = (index, shouldPlay = true) => {
                if (index >= timelineClips.length) return stopTimeline();
                selectClip(timelineClips[index].instanceId, shouldPlay, true);
            };
            previewPlayer.addEventListener('timeupdate', () => {
                if (!isTimelinePlaying) return;
                const currentClip = timelineClips[currentTimelineIndex];
                if (currentClip && previewPlayer.currentTime >= currentClip.endTrim) {
                    previewPlayer.pause();
                    currentTimelineIndex++;
                    playClip(currentTimelineIndex);
                }
            });
            const updatePlayheadPosition = () => {
                if (!isTimelinePlaying) return;
                let timeOfPreviousClips = 0;
                for (let i = 0; i < currentTimelineIndex; i++) {
                    timeOfPreviousClips += timelineClips[i].clipDuration;
                }
                const currentClip = timelineClips[currentTimelineIndex];
                if (!currentClip) return stopTimeline();
                const currentTotalTime = timeOfPreviousClips + (previewPlayer.currentTime - currentClip.startTrim);
                updatePlayhead(currentTotalTime * PIXELS_PER_SECOND);
                playheadAnimationId = requestAnimationFrame(updatePlayheadPosition);
            };
            window.selectClip = (instanceId, shouldPlay = false, partOfSequence = false, startTimeOverride = null) => {
                const clip = timelineClips.find(c => c.instanceId === instanceId);
                if (clip) {
                    if (previewPlayer.src !== clip.url) previewPlayer.src = clip.url;
                    previewPlayer.currentTime = startTimeOverride !== null ? startTimeOverride : clip.startTrim;
                    if(shouldPlay) previewPlayer.play();
                    previewPlayer.classList.remove('hidden');
                    previewPlaceholder.classList.add('hidden');
                    document.querySelectorAll('.timeline-clip').forEach(el => el.classList.remove('selected'));
                    const selectedEl = document.querySelector(`[data-instance-id="${instanceId}"]`);
                    if(selectedEl) selectedEl.classList.add('selected');
                    selectedClipInstanceId = instanceId;
                    timelineSplitBtn.disabled = false;
                    timelineSplitBtn.classList.remove('btn-disabled');
                    timelineDeleteClipBtn.disabled = false;
                    timelineDeleteClipBtn.classList.remove('btn-disabled');
                    if (!partOfSequence) {
                        pauseTimeline();
                        currentTimelineIndex = timelineClips.findIndex(c => c.instanceId === instanceId);
                        let timeOfPreviousClips = 0;
                        for (let i = 0; i < currentTimelineIndex; i++) {
                            timeOfPreviousClips += timelineClips[i].clipDuration;
                        }
                        updatePlayhead(timeOfPreviousClips * PIXELS_PER_SECOND);
                    }
                }
            };
            timelineExportBtn.addEventListener('click', async () => { /* ... */ });
            mediaUpload.addEventListener('change', (e) => {
                const files = Array.from(e.target.files);
                Promise.all(files.map(file => new Promise((resolve) => {
                    if (file.type.startsWith('video/')) {
                        const videoEl = document.createElement('video');
                        videoEl.src = URL.createObjectURL(file);
                        videoEl.onloadedmetadata = () => {
                            mediaBin.push({ id: file.name, file: file, url: videoEl.src, name: file.name, duration: videoEl.duration });
                            resolve();
                        };
                    } else resolve();
                }))).then(renderMediaBin);
            });
            const scrub = (e, targetElement) => {
                pauseTimeline();
                const timelineRect = targetElement.getBoundingClientRect();
                const scrollLeft = timelineContainer.scrollLeft;
                let x = e.clientX - timelineRect.left + (targetElement === timelineRulerContainer ? 0 : timelineTrack.getBoundingClientRect().left - timelineRect.left) + scrollLeft;
                x = Math.max(0, x);
                const globalTime = x / PIXELS_PER_SECOND;
                const { clip, localTime, clipIndex } = findClipAtGlobalTime(globalTime);
                if (clip) {
                    if (previewPlayer.src !== clip.url) previewPlayer.src = clip.url;
                    previewPlayer.currentTime = clip.startTrim + localTime;
                    currentTimelineIndex = clipIndex;
                    updatePlayhead(x);
                }
            };
            const findClipAtGlobalTime = (globalTime) => {
                let cumulativeTime = 0;
                for (const [index, clip] of timelineClips.entries()) {
                    if (globalTime >= cumulativeTime && globalTime < cumulativeTime + clip.clipDuration) {
                        return { clip, localTime: globalTime - cumulativeTime, clipIndex: index };
                    }
                    cumulativeTime += clip.clipDuration;
                }
                const lastClip = timelineClips[timelineClips.length - 1];
                return { clip: lastClip, localTime: lastClip?.clipDuration || 0, clipIndex: timelineClips.length - 1 };
            };
            const updatePlayhead = (pixelX) => {
                const playheadEl = document.getElementById('playhead');
                if (playheadEl) playheadEl.style.transform = `translateX(${pixelX}px)`;
            };
            timelineRulerContainer.addEventListener('mousedown', (e) => { isScrubbing = true; scrub(e, timelineRulerContainer); });
            document.addEventListener('mousemove', (e) => { if (isScrubbing) scrub(e, timelineRulerContainer); });
            document.addEventListener('mouseup', () => { isScrubbing = false; });
            const splitSelectedClip = () => {
                if (!selectedClipInstanceId || isTimelinePlaying) return;
                const playheadEl = document.getElementById('playhead');
                const playheadX = parseFloat((playheadEl.style.transform || 'translateX(0px)').replace(/[^0-9.-]/g, ''));
                const globalSplitTime = playheadX / PIXELS_PER_SECOND;
                let cumulativeTime = 0, targetClipIndex = -1, targetClip = null;
                for (const [index, clip] of timelineClips.entries()) {
                    if (clip.instanceId === selectedClipInstanceId) {
                        targetClipIndex = index;
                        targetClip = clip;
                        break;
                    }
                    cumulativeTime += clip.clipDuration;
                }
                if (!targetClip) return;
                const localSplitTime = globalSplitTime - cumulativeTime;
                if (localSplitTime <= 0.1 || localSplitTime >= targetClip.clipDuration - 0.1) return alert('Tidak bisa memotong di awal atau akhir klip.');
                const originalEndTrim = targetClip.endTrim;
                targetClip.endTrim = targetClip.startTrim + localSplitTime;
                targetClip.clipDuration = targetClip.endTrim - targetClip.startTrim;
                const newClip = { ...targetClip, instanceId: `clip-${Date.now()}`, startTrim: targetClip.endTrim, endTrim: originalEndTrim, clipDuration: originalEndTrim - targetClip.endTrim };
                timelineClips.splice(targetClipIndex + 1, 0, newClip);
                renderTimeline();
            };
            timelineSplitBtn.addEventListener('click', splitSelectedClip);
            const startTrimming = (e, side) => {
                e.stopPropagation();
                isTrimming = true;
                trimmingHandleSide = side;
                document.body.style.cursor = 'ew-resize';
            };
            const doTrim = (e) => {
                if (!isTrimming || !selectedClipInstanceId) return;
                const clipIndex = timelineClips.findIndex(c => c.instanceId === selectedClipInstanceId);
                const clip = timelineClips[clipIndex];
                if (!clip) return;
                const timelineRect = timelineTrack.getBoundingClientRect();
                const scrollLeft = timelineContainer.scrollLeft;
                const mouseX = e.clientX - timelineRect.left + scrollLeft;
                let clipStartPixel = 0;
                for(let i=0; i<clipIndex; i++) clipStartPixel += timelineClips[i].clipDuration * PIXELS_PER_SECOND;
                const timeInClip = (mouseX - clipStartPixel) / PIXELS_PER_SECOND;

                if (trimmingHandleSide === 'left') {
                    const delta = timeInClip;
                    const newStartTrim = clip.startTrim + delta;
                    if (newStartTrim >= clip.endTrim - 0.2 || newStartTrim < 0) return;
                    clip.clipDuration -= delta;
                    clip.startTrim = newStartTrim;
                } else if (trimmingHandleSide === 'right') {
                    const newDuration = timeInClip;
                    if (newDuration < 0.2 || clip.startTrim + newDuration > clip.duration) return;
                    clip.clipDuration = newDuration;
                    clip.endTrim = clip.startTrim + newDuration;
                }
                renderTimeline();
                const selectedEl = document.querySelector(`[data-instance-id="${selectedClipInstanceId}"]`);
                if(selectedEl) selectedEl.classList.add('selected');
            };
            const stopTrimming = () => {
                isTrimming = false;
                trimmingHandleSide = null;
                document.body.style.cursor = 'default';
            };
            document.addEventListener('mousemove', doTrim);
            document.addEventListener('mouseup', stopTrimming);

            const formatTimeForRuler = (seconds) => {
                const minutes = Math.floor(seconds / 60);
                const secs = Math.floor(seconds % 60);
                return `${String(minutes).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
            };
            const renderTimelineRuler = () => {
                const totalWidth = totalTimelineDuration * PIXELS_PER_SECOND;
                timelineRulerContainer.innerHTML = ''; // Hapus ruler lama
                const rulerWrapper = document.createElement('div');
                rulerWrapper.style.width = `${totalWidth}px`;
                rulerWrapper.className = 'h-full relative';
                for (let i = 0; i <= totalTimelineDuration; i++) {
                    const tick = document.createElement('div');
                    tick.className = 'ruler-tick';
                    tick.style.left = `${i * PIXELS_PER_SECOND}px`;
                    if (i % 5 === 0) {
                        tick.classList.add('major');
                        const label = document.createElement('span');
                        label.className = 'ruler-label';
                        label.textContent = formatTimeForRuler(i);
                        label.style.left = `${i * PIXELS_PER_SECOND}px`;
                        rulerWrapper.appendChild(label);
                    } else {
                        tick.classList.add('minor');
                    }
                    rulerWrapper.appendChild(tick);
                }
                timelineRulerContainer.appendChild(rulerWrapper);
            };
            timelineContainer.addEventListener('scroll', () => { if(timelineRulerContainer) timelineRulerContainer.scrollLeft = timelineContainer.scrollLeft; });
            if (timelineDeleteClipBtn) timelineDeleteClipBtn.addEventListener('click', deleteSelectedClip);

            // --- Inisialisasi Awal ---
            showPage('home');
            setStatus('empty');
            updateButtonsState();
            lucide.createIcons();
        });
    </script>
</body>
</html>

