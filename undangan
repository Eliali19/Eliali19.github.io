<!DOCTYPE html>
<html lang="id" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Wedding of Ally & Elia</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts for Classic/Elegant Look: Great Vibes (Script), Cormorant Garamond (Serif Elegant), Inter (Body) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@400;700&family=Great+Vibes&family=Inter:wght@300;400;600&display=swap" rel="stylesheet">

    <!-- Firebase SDK Imports for RSVP and Guestbook (Firestore) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, onSnapshot, collection, query, serverTimestamp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // --- FIREBASE INITIALIZATION AND AUTHENTICATION ---
        
        // Mandatory Canvas Variables (provided by the environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        if (firebaseConfig) {
            // Initialize Firebase services
            const app = initializeApp(firebaseConfig);
            const db = getFirestore(app);
            const auth = getAuth(app);
            setLogLevel('Debug'); // Enable Firebase Debug Logging

            // Make Firebase instances and variables globally accessible for event listeners
            window.db = db;
            window.auth = auth;
            window.appId = appId;
            window.serverTimestamp = serverTimestamp;

            // Handle Authentication State
            onAuthStateChanged(auth, async (user) => {
                let userId;
                if (user) {
                    userId = user.uid;
                } else {
                    // Sign in using custom token or anonymously if token is missing
                    if (initialAuthToken) {
                        try {
                            await signInWithCustomToken(auth, initialAuthToken);
                            userId = auth.currentUser.uid;
                        } catch (error) {
                            console.error("Error signing in with custom token, signing in anonymously:", error);
                            await signInAnonymously(auth);
                            userId = auth.currentUser.uid;
                        }
                    } else {
                        await signInAnonymously(auth);
                        userId = auth.currentUser.uid;
                    }
                }
                
                window.userId = userId;
                console.log("Firestore ready. User ID:", userId);

                // Start real-time listening for guestbook entries
                loadGuestbookEntries(db, appId);
            });

            // --- FIRESTORE CRUD FUNCTIONS ---

            // Function to save RSVP (Private Data)
            window.saveRsvp = async (guestName, attendance, message) => {
                if (!window.userId) return { success: false, message: "User not authenticated yet. Please wait a moment." };
                const collectionPath = `artifacts/${appId}/users/${window.userId}/rsvps`;
                try {
                    await addDoc(collection(db, collectionPath), {
                        guestName: guestName,
                        attendance: attendance,
                        message: message,
                        timestamp: serverTimestamp(),
                        userId: window.userId
                    });
                    return { success: true, message: "Konfirmasi kehadiran berhasil disimpan. Terima kasih!" };
                } catch (e) {
                    console.error("Error saving RSVP:", e);
                    return { success: false, message: "Gagal menyimpan RSVP: " + e.message };
                }
            };

            // Function to save Guestbook Entry (Public Data)
            window.saveGuestbookEntry = async (name, message) => {
                if (!window.userId) return { success: false, message: "User not authenticated yet. Please wait a moment." };
                const collectionPath = `artifacts/${appId}/public/data/guestbook_entries`;
                try {
                    await addDoc(collection(db, collectionPath), {
                        name: name,
                        message: message,
                        timestamp: serverTimestamp(),
                        userId: window.userId
                    });
                    return { success: true, message: "Ucapan berhasil dikirim. Terima kasih!" };
                } catch (e) {
                    console.error("Error saving guestbook entry:", e);
                    return { success: false, message: "Gagal menyimpan ucapan: " + e.message };
                }
            };

            // Function to load Guestbook Entries in real-time
            function loadGuestbookEntries(db, appId) {
                const collectionPath = `artifacts/${appId}/public/data/guestbook_entries`;
                const q = query(collection(db, collectionPath));

                onSnapshot(q, (snapshot) => {
                    const entries = [];
                    snapshot.forEach((doc) => {
                        entries.push({ id: doc.id, ...doc.data() });
                    });
                    // Sort by timestamp (descending - newest first) in memory
                    entries.sort((a, b) => (b.timestamp?.seconds || 0) - (a.timestamp?.seconds || 0));
                    
                    const container = document.getElementById('guestbook-list');
                    if (container) {
                        container.innerHTML = ''; // Clear existing entries

                        if (entries.length === 0) {
                            container.innerHTML = '<p class="text-center text-gray-500 italic">Belum ada ucapan. Jadilah yang pertama!</p>';
                        }

                        entries.forEach(entry => {
                            const date = entry.timestamp ? new Date(entry.timestamp.seconds * 1000).toLocaleDateString('id-ID', { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' }) : 'Baru saja';
                            const entryHtml = `
                                <div class="bg-white/80 backdrop-blur-sm p-4 rounded-xl shadow-md mb-4 border border-blush-dark/30 animate-fade-in">
                                    <p class="font-serif-elegant text-lg font-semibold text-blush-dark">${entry.name}</p>
                                    <p class="text-xs text-gray-500 mb-2">${date}</p>
                                    <p class="text-gray-700 mt-2 whitespace-pre-wrap">${entry.message}</p>
                                    <p class="text-right text-xs text-gray-400 mt-2">ID User: ${entry.userId.substring(0, 8)}...</p>
                                </div>
                            `;
                            container.insertAdjacentHTML('beforeend', entryHtml);
                        });
                    }
                }, (error) => {
                    console.error("Error loading guestbook entries:", error);
                    const container = document.getElementById('guestbook-list');
                    if (container) {
                        container.innerHTML = '<p class="text-center text-red-500">Gagal memuat ucapan. Silakan coba muat ulang halaman.</p>';
                    }
                });
            }
        } else {
            console.warn("Firebase config not available. RSVP and Guestbook features will not be active.");
        }
    </script>

    <!-- --- CUSTOM STYLES & TAILWIND CONFIGURATION --- -->
    <style>
        /* Define Custom Color Palette and Fonts */
        :root {
            --color-blush-light: #FFEDED; /* Softest Blush */
            --color-blush-medium: #F4CCCC; /* Dominant Blush */
            --color-blush-dark: #D4A3AE; /* Accent Blush - Classic/Elegant */
            --color-text-primary: #3d3436; /* Dark Brown/Maroon for contrast */
        }
        
        @layer utilities {
            /* Custom Colors */
            .bg-blush-light { background-color: var(--color-blush-light); }
            .bg-blush-medium { background-color: var(--color-blush-medium); }
            .bg-blush-dark { background-color: var(--color-blush-dark); }
            .text-blush-dark { color: var(--color-blush-dark); }
            .text-text-primary { color: var(--color-text-primary); }
            .border-blush-dark { border-color: var(--color-blush-dark); }
            /* Custom Fonts */
            .font-script { font-family: 'Great Vibes', cursive; }
            .font-serif-elegant { font-family: 'Cormorant Garamond', serif; }
            .font-sans-body { font-family: 'Inter', sans-serif; }
        }

        /* Base styles */
        body {
            @apply font-sans-body bg-blush-light text-text-primary overflow-x-hidden;
        }

        /* Custom Scrollbar for subtle look */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-thumb { background-color: var(--color-blush-dark); border-radius: 4px; }
        ::-webkit-scrollbar-track { background-color: var(--color-blush-light); }

        /* Smooth Animation for Scroll (Fade In) */
        .animate-fade-in {
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 1s cubic-bezier(0.25, 0.46, 0.45, 0.94), transform 1s cubic-bezier(0.25, 0.46, 0.45, 0.94); /* Smooth Cubic Bezier */
        }
        .is-visible {
            opacity: 1;
            transform: translateY(0);
        }

        /* Parallax Background Effect for Hero Section */
        #hero-section {
            background-attachment: fixed; /* Parallax effect */
            background-position: center;
            background-repeat: no-repeat;
            background-size: cover;
        }

        /* Button Styling */
        .btn-primary {
            @apply w-full transition duration-300 ease-in-out py-3 rounded-full font-bold shadow-lg hover:shadow-xl focus:outline-none focus:ring-4 focus:ring-blush-dark/50;
        }

        .btn-blush {
            @apply bg-blush-dark text-white hover:bg-[#b07f8b]; /* Slightly darker hover */
        }
    </style>
</head>
<body>

    <!-- --- MUSIC PLAYER & CONTROL --- -->
    <audio id="background-music" loop>
        <!-- Placeholder Audio: User must replace this with a direct link to an MP3 file (e.g., hosted on Google Drive, Dropbox, or a free host with direct link). -->
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-slow-sweep-transition-1100.mp3" type="audio/mp3">
        Your browser does not support the audio element.
    </audio>
    <div id="music-control" class="fixed bottom-4 right-4 z-50 p-3 bg-blush-dark/70 rounded-full shadow-lg cursor-pointer hidden transition duration-300 hover:scale-110">
        <!-- Speaker Icon (Default: ON) -->
        <svg id="music-icon" class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 24 24">
            <path d="M14 3.23l2.83 2.83-2.83 2.83V3.23zM2 9h4l5-5v16l-5-5H2V9zm18.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM17 12c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02z"/>
        </svg>
    </div>

    <!-- 1. GATEKEEPER (Initial Screen/Cover) -->
    <div id="gatekeeper" class="fixed inset-0 z-[100] bg-blush-medium flex flex-col items-center justify-center p-6 text-center transition-opacity duration-1000 ease-in-out">
        <p class="text-sm font-sans-body mb-2 text-text-primary/70">Undangan Pernikahan</p>
        <h1 class="text-7xl sm:text-8xl font-script text-blush-dark mb-4 drop-shadow-md">Ally & Elia</h1>
        <p class="text-xl font-serif-elegant mb-8 text-text-primary">19 Mei 2026</p>
        
        <div class="w-full max-w-sm">
            <!-- Tamu Undangan Target -->
            <p class="text-sm text-text-primary/80 mb-2">Kepada Yth:</p>
            <p id="guest-target" class="text-2xl font-serif-elegant font-bold text-text-primary mb-12 truncate max-w-full">[Nama Tamu]</p>
            <!-- Tombol Buka Undangan -->
            <button id="open-invitation-btn" class="btn-primary btn-blush">
                Buka Undangan
                <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 inline ml-2" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M19 12H5"/><path d="m12 5-7 7 7 7"/></svg>
            </button>
            <p class="mt-4 text-xs text-text-primary/60">Tekan tombol di atas untuk melihat detail undangan.</p>
        </div>
    </div>

    <!-- 2. MAIN INVITATION CONTENT -->
    <div id="invitation" class="min-h-screen hidden">
        
        <!-- 2.1 HERO SECTION (Cover) - Parallax Ready -->
        <section id="hero-section" class="min-h-screen flex flex-col justify-center items-center text-white p-6 relative z-10" style="background-image: url('https://lh3.googleusercontent.com/d/1x7RuSzlgmENAj3fO4Qq9No2DkfMF17Hr');">
            <div class="absolute inset-0 bg-black/40 z-0"></div>
            <div class="relative z-10 text-center animate-fade-in is-visible">
                <p class="text-lg sm:text-xl font-serif-elegant tracking-widest mb-4">THE WEDDING OF</p>
                <h1 class="text-7xl sm:text-8xl font-script mb-6 drop-shadow-lg">Ally & Elia</h1>
                <p class="text-xl sm:text-2xl font-serif-elegant mb-8">19 Mei 2026</p>
                <a href="#ceremony" class="inline-block px-8 py-3 bg-white/20 backdrop-blur-sm rounded-full text-white font-bold text-sm uppercase tracking-wider hover:bg-white/40 transition duration-300">Lihat Detail Acara</a>
            </div>
        </section>

        <!-- 2.2 PEMBUKAAN / GUESTBOOK OPENER -->
        <section class="py-16 px-6 text-center">
            <div class="max-w-xl mx-auto animate-fade-in">
                <p class="text-2xl font-serif-elegant text-blush-dark mb-6">Assalamu'alaikum Warahmatullahi Wabarakatuh</p>
                <p class="text-text-primary text-md leading-relaxed whitespace-pre-wrap font-sans-body">
                    Bismillahirahmanirrahim.
                    Tanpa mengurangi rasa hormat, perkenankan kami mengundang Bapak/Ibu/Saudara/i sekalian, untuk menghadiri acara pernikahan kami :
                    
                    <span class="block mt-4 text-3xl font-serif-elegant font-bold text-blush-dark/90">Nur Ali & Eliasari mailani S.Keb</span>
                </p>
            </div>
        </section>

        <!-- 2.3 COUNTDOWN & NAMES -->
        <section class="py-16 bg-blush-medium/50 text-center shadow-inner">
             <div class="max-w-3xl mx-auto px-6">
                <h2 class="text-4xl font-script text-blush-dark mb-8 animate-fade-in">Mempelai</h2>
                <div class="flex flex-col md:flex-row justify-center items-center space-y-12 md:space-y-0 md:space-x-16">
                    <!-- Groom -->
                    <div class="animate-fade-in delay-200">
                        <h3 class="text-6xl font-script text-text-primary mb-2">Ally</h3>
                        <p class="text-lg font-serif-elegant text-text-primary/80">Putra dari Bpk. [Nama Ayah] & Ibu. [Nama Ibu]</p>
                    </div>

                    <p class="text-6xl font-script text-blush-dark">&</p>

                    <!-- Bride -->
                    <div class="animate-fade-in delay-400">
                        <h3 class="text-6xl font-script text-text-primary mb-2">Elia</h3>
                        <p class="text-lg font-serif-elegant text-text-primary/80">Putri dari Bpk. [Nama Ayah] & Ibu. [Nama Ibu]</p>
                    </div>
                </div>

                <!-- Countdown Timer -->
                <div class="mt-16 animate-fade-in delay-600">
                    <h3 class="text-2xl font-serif-elegant text-blush-dark mb-4">Menuju Hari Bahagia</h3>
                    <div id="countdown" class="flex justify-center space-x-4 sm:space-x-8 text-text-primary">
                        <div class="flex flex-col items-center">
                            <span class="text-4xl sm:text-5xl font-bold font-serif-elegant" id="days">00</span>
                            <span class="text-xs sm:text-sm uppercase tracking-wider mt-1">Hari</span>
                        </div>
                        <div class="flex flex-col items-center">
                            <span class="text-4xl sm:text-5xl font-bold font-serif-elegant" id="hours">00</span>
                            <span class="text-xs sm:text-sm uppercase tracking-wider mt-1">Jam</span>
                        </div>
                        <div class="flex flex-col items-center">
                            <span class="text-4xl sm:text-5xl font-bold font-serif-elegant" id="minutes">00</span>
                            <span class="text-xs sm:text-sm uppercase tracking-wider mt-1">Menit</span>
                        </div>
                        <div class="flex flex-col items-center">
                            <span class="text-4xl sm:text-5xl font-bold font-serif-elegant" id="seconds">00</span>
                            <span class="text-xs sm:text-sm uppercase tracking-wider mt-1">Detik</span>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- 2.4 CEREMONY & RECEPTION DETAILS -->
        <section id="ceremony" class="py-16 px-6 text-center bg-blush-light">
            <h2 class="text-4xl font-script text-blush-dark mb-12 animate-fade-in">Detail Acara</h2>

            <div class="flex flex-col max-w-4xl mx-auto space-y-10">
                
                <!-- Akad Nikah -->
                <div class="flex-1 bg-white p-8 rounded-xl shadow-xl border-t-4 border-blush-dark animate-fade-in delay-200">
                    <h3 class="text-3xl font-serif-elegant font-bold text-text-primary mb-4">Akad Nikah</h3>
                    <p class="text-sm uppercase tracking-widest text-blush-dark mb-4">Selasa, 19 Mei 2026</p>
                    <p class="text-xl font-serif-elegant mb-4">Pukul 09:16 WIB</p>
                    <p class="text-gray-600 mb-6">Kediaman Mempelai Wanita</p>
                    <!-- Google Maps Embed for Akad (Placeholder: User must change to the actual map embed URL for "Kediaman Mempelai Wanita") -->
                    <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3966.19532654876!2d106.812236!3d-6.230756!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x2e69f37c385f0967%3A0xc3926e849c67d712!2sMonas!5e0!3m2!1sen!2sid!4v1628588888888!5m2!1sen!2sid" width="100%" height="300" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade" class="rounded-lg shadow-md mb-4"></iframe>
                    <a href="https://maps.app.goo.gl/" target="_blank" class="text-sm font-semibold text-blush-dark hover:underline">Lihat di Google Maps</a>
                </div>

                <!-- Resepsi -->
                <div class="flex-1 bg-white p-8 rounded-xl shadow-xl border-t-4 border-blush-dark animate-fade-in delay-400">
                    <h3 class="text-3xl font-serif-elegant font-bold text-text-primary mb-4">Resepsi</h3>
                    <p class="text-sm uppercase tracking-widest text-blush-dark mb-4">Selasa, 19 Mei 2026</p>
                    <p class="text-xl font-serif-elegant mb-4">Pukul 10:06 WIB</p>
                    <p class="text-gray-600 mb-6">Kp.Gabus Bulak</p>
                     <!-- Google Maps Embed for Resepsi (Placeholder: User must change to the actual map embed URL for "Kp.Gabus Bulak") -->
                    <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3966.302381286953!2d106.8159146!3d-6.216669!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x2e69f5e1f0b0c9f1%3A0x6b4c1c9b0e1b2f0a!2sStasiun%20Gambir!5e0!3m2!1sen!2sid!4v1628588888888!5m2!1sen!2sid" width="100%" height="300" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade" class="rounded-lg shadow-md mb-4"></iframe>
                    <a href="https://maps.app.goo.gl/" target="_blank" class="text-sm font-semibold text-blush-dark hover:underline">Lihat di Google Maps</a>
                </div>
            </div>
        </section>
        
        <!-- 2.5 GALLERY SECTION -->
        <section class="py-16 px-6 text-center">
            <h2 class="text-4xl font-script text-blush-dark mb-12 animate-fade-in">Galeri Foto</h2>
            <div id="photo-gallery" class="grid grid-cols-2 md:grid-cols-3 gap-4 max-w-6xl mx-auto">
                <img src="https://lh3.googleusercontent.com/d/1vLoS9O9jw9BWLzrL6oHjTlhcm1ws21nQ" alt="Foto Pasangan 1" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in">
                <img src="https://lh3.googleusercontent.com/d/1HIRCuUjnv1-kDjoI3ugu1iSl8RPLdo5V" alt="Foto Pasangan 2" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in delay-100">
                <img src="https://lh3.googleusercontent.com/d/1K6jZ7vlKEKfjKiMsDySdO8ookgM9HPoK" alt="Foto Pasangan 3" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in delay-200">
                <img src="https://lh3.googleusercontent.com/d/17aPB-M3sqDuAS4Fu4WiDn9yQfkXZrdjr" alt="Foto Pasangan 4" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in delay-300">
                <img src="https://lh3.googleusercontent.com/d/1u2_5LsFD_MvAZSg_EOuQE-wNwCTw8zvE" alt="Foto Pasangan 5" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in delay-400">
                <img src="https://lh3.googleusercontent.com/d/1YMJh9eEjnmGVMhrhejl5d9C3w-whj4NZ" alt="Foto Pasangan 6" class="w-full h-auto rounded-xl shadow-lg object-cover aspect-square animate-fade-in delay-500">
            </div>
        </section>

        <!-- 2.6 OUR STORY (Cerita Singkat) -->
        <section class="py-16 bg-blush-medium/50 px-6 text-center">
            <h2 class="text-4xl font-script text-blush-dark mb-8 animate-fade-in">Our Story</h2>
            <div class="max-w-xl mx-auto animate-fade-in delay-200">
                <p class="text-text-primary text-lg font-serif-elegant italic leading-relaxed">
                    "cerita nya panjang susah jelasinnya"
                </p>
            </div>
        </section>

        <!-- 2.7 RSVP FORM -->
        <section id="rsvp" class="py-16 px-6 bg-blush-light">
            <div class="max-w-xl mx-auto text-center animate-fade-in">
                <h2 class="text-4xl font-script text-blush-dark mb-8">Konfirmasi Kehadiran (RSVP)</h2>
                <form id="rsvp-form" class="bg-white p-6 md:p-8 rounded-xl shadow-2xl border-t-4 border-blush-dark">
                    <div class="mb-4 text-left">
                        <label for="rsvp-name" class="block text-sm font-medium text-gray-700 mb-1">Nama Anda:</label>
                        <input type="text" id="rsvp-name" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-blush-dark focus:border-blush-dark transition duration-200">
                    </div>
                    <div class="mb-6 text-left">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Konfirmasi Kehadiran:</label>
                        <div class="flex items-center space-x-6">
                            <label class="inline-flex items-center">
                                <input type="radio" name="attendance" value="Hadir" required class="form-radio h-4 w-4 text-blush-dark border-gray-300" checked>
                                <span class="ml-2 text-gray-700">Hadir</span>
                            </label>
                            <label class="inline-flex items-center">
                                <input type="radio" name="attendance" value="Tidak Hadir" class="form-radio h-4 w-4 text-blush-dark border-gray-300">
                                <span class="ml-2 text-gray-700">Tidak Hadir</span>
                            </label>
                        </div>
                    </div>
                     <div class="mb-6 text-left">
                        <label for="rsvp-message" class="block text-sm font-medium text-gray-700 mb-1">Pesan (Opsional):</label>
                        <textarea id="rsvp-message" rows="3" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-blush-dark focus:border-blush-dark transition duration-200"></textarea>
                    </div>

                    <button type="submit" class="btn-primary btn-blush">Kirim Konfirmasi</button>
                    <div id="rsvp-message-box" class="mt-4 p-3 rounded-lg text-sm hidden"></div>
                </form>
            </div>
        </section>
        
        <!-- 2.8 GUESTBOOK / UCAPAN & DOA -->
        <section id="guestbook" class="py-16 px-6 bg-blush-medium">
            <div class="max-w-xl mx-auto animate-fade-in">
                <h2 class="text-4xl font-script text-blush-dark mb-8 text-center">Ucapan & Doa</h2>
                
                <!-- Guestbook Form -->
                <form id="guestbook-form" class="bg-white p-6 md:p-8 rounded-xl shadow-2xl border-t-4 border-blush-dark mb-10">
                    <div class="mb-4 text-left">
                        <label for="guestbook-name" class="block text-sm font-medium text-gray-700 mb-1">Nama Anda:</label>
                        <input type="text" id="guestbook-name" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-blush-dark focus:border-blush-dark transition duration-200">
                    </div>
                    <div class="mb-6 text-left">
                        <label for="guestbook-message" class="block text-sm font-medium text-gray-700 mb-1">Ucapan & Doa:</label>
                        <textarea id="guestbook-message" rows="4" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-blush-dark focus:border-blush-dark transition duration-200"></textarea>
                    </div>

                    <button type="submit" class="btn-primary btn-blush">Kirim Ucapan</button>
                    <div id="guestbook-message-box" class="mt-4 p-3 rounded-lg text-sm hidden"></div>
                </form>

                <!-- Guestbook List -->
                <h3 class="text-2xl font-serif-elegant text-text-primary mb-6 text-center">Kiriman Doa</h3>
                <div id="guestbook-list" class="space-y-4 max-h-96 overflow-y-auto p-4 border rounded-xl bg-blush-light/50 border-blush-dark/50 shadow-inner">
                    <!-- Entries will be dynamically loaded here by Firebase onSnapshot -->
                    <p class="text-center text-gray-500 italic">Memuat ucapan...</p>
                </div>
            </div>
        </section>

        <!-- 2.9 CLOSING -->
        <footer class="py-10 px-6 text-center bg-text-primary text-white">
            <p class="text-xl font-serif-elegant mb-2">Dengan segala kerendahan hati,</p>
            <h2 class="text-6xl font-script text-blush-dark mb-4">Ally & Elia</h2>
            <p class="text-sm font-serif-elegant mb-2">Merupakan suatu kehormatan dan kebahagiaan bagi kami apabila Bapak/Ibu/Saudara/i berkenan hadir dan memberikan doa restu.</p>
            <p class="text-xs mt-4 text-gray-400">© 2026 The Wedding of Ally & Elia. Dibuat dengan cinta.</p>
        </footer>

    </div>
    
    <!-- --- MAIN JAVASCRIPT LOGIC --- -->
    <script>
        // --- CONSTANTS ---
        // Target wedding date and time
        const WEDDING_DATE = new Date("2026-05-19T09:16:00").getTime(); 
        
        // --- CORE APPLICATION STATE & LOGIC ---

        document.addEventListener('DOMContentLoaded', () => {
            const gatekeeper = document.getElementById('gatekeeper');
            const invitation = document.getElementById('invitation');
            const openBtn = document.getElementById('open-invitation-btn');
            const musicControl = document.getElementById('music-control');
            const audioPlayer = document.getElementById('background-music');
            
            // 1. Initial Guest Name from URL
            const urlParams = new URLSearchParams(window.location.search);
            // Check for 'to' parameter (e.g., ?to=Bapak+Fajar)
            const guestName = urlParams.get('to') || 'Tamu Undangan'; 
            document.getElementById('guest-target').textContent = decodeURIComponent(guestName.replace(/\+/g, ' '));
            
            // 2. Gatekeeper Logic (Open Invitation Button)
            openBtn.addEventListener('click', () => {
                // Smoothly hide gatekeeper screen
                gatekeeper.style.opacity = '0';
                
                setTimeout(() => {
                    gatekeeper.classList.add('hidden');
                    invitation.classList.remove('hidden');
                    musicControl.classList.remove('hidden');
                    
                    // Attempt to play music on user interaction
                    audioPlayer.volume = 0.5;
                    const playPromise = audioPlayer.play();
                    
                    if (playPromise !== undefined) {
                        playPromise.then(() => {
                            console.log("Audio started successfully.");
                        }).catch(error => {
                            // Penanganan AbortError: Ini adalah error umum ketika browser mencegah autoplay
                            // Kita hanya mencatatnya untuk menghindari unhandled rejection.
                            if (error.name === 'AbortError') {
                                console.log("Autoplay dicegah oleh browser (AbortError). Kontrol musik tersedia.");
                            } else {
                                console.error("Autoplay dicegah atau gagal:", error);
                            }
                            // Kontrol musik sudah ditampilkan, jadi biarkan pengguna mengontrol secara manual.
                        });
                    }
                    
                    // Start general scroll animations
                    setupScrollAnimations();
                }, 800);
            });

            // 3. Countdown Timer Function
            const updateCountdown = () => {
                const now = new Date().getTime();
                const distance = WEDDING_DATE - now;

                // Calculate days, hours, minutes, and seconds
                const days = Math.floor(distance / (1000 * 60 * 60 * 24));
                const hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
                const minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
                const seconds = Math.floor((distance % (1000 * 60)) / 1000);

                if (distance < 0) {
                    const cD = document.getElementById('countdown');
                    if (cD) cD.innerHTML = '<p class="text-xl text-blush-dark font-serif-elegant font-bold">Acara Telah Berlangsung</p>';
                    clearInterval(window.countdownInterval);
                    return;
                }

                // Update DOM elements
                const update = (id, value) => {
                    const el = document.getElementById(id);
                    if (el) el.textContent = String(value).padStart(2, '0');
                };

                update('days', days);
                update('hours', hours);
                update('minutes', minutes);
                update('seconds', seconds);
            };

            updateCountdown(); // Initial call
            window.countdownInterval = setInterval(updateCountdown, 1000);

            // 4. Music Control Logic
            musicControl.addEventListener('click', () => {
                if (audioPlayer.paused) {
                    // Coba putar audio dan tangani potensi AbortError saat play()
                    const playPromise = audioPlayer.play();
                    if (playPromise !== undefined) {
                        playPromise.then(() => {
                            // Berhasil diputar
                            musicControl.innerHTML = '<svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 24 24"><path d="M14 3.23l2.83 2.83-2.83 2.83V3.23zM2 9h4l5-5v16l-5-5H2V9zm18.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM17 12c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02z"/></svg>'; 
                        }).catch(error => {
                            // Tangani AbortError jika terjadi
                            if (error.name !== 'AbortError') {
                                console.error("Gagal memutar audio saat tombol diklik:", error);
                            }
                            // Icon tetap mati karena play gagal
                            // Tidak perlu mengganti icon jika play gagal
                        });
                    }
                } else {
                    audioPlayer.pause();
                    // Change icon to speaker OFF (Placeholder SVG)
                    musicControl.innerHTML = '<svg class="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 24 24"><path d="M16.5 12c0-1.77-1.02-3.29-2.5-4.03v2.21l2.45 2.45c.03-.2.05-.41.05-.63zm2.5 0c0 .94-.2 1.82-.54 2.64l1.51 1.51C20.63 14.62 21 13.38 21 12c0-4.28-2.99-7.86-7-8.77v2.06c2.89.86 5 3.54 5 6.71zM4.34 3L2.93 4.41l5.19 5.19-2.09 2.09c0 .35.05.69.1 1.02l-1.51 1.51C3.81 14.28 3.5 13.17 3.5 12c0-1.92.54-3.71 1.48-5.27L2.93 4.41 4.34 3zm1.48 4.32l2.35 2.35L9 12.37V9H6.94l-2.02-2.02zM10.86 16.95l1.09 1.09c.47-.28.98-.56 1.5-.83v-1.74l-1.57-1.57zm.14-11.2V4.07c.84.17 1.67.49 2.44.97l-1.51 1.51c-.69-.43-1.42-.7-2.2-.82z"/></svg>';
                }
            });

            // 5. RSVP Form Submission (Uses global saveRsvp from the module script)
            document.getElementById('rsvp-form').addEventListener('submit', async (e) => {
                e.preventDefault();
                const name = document.getElementById('rsvp-name').value.trim();
                const attendance = document.querySelector('input[name="attendance"]:checked').value;
                const message = document.getElementById('rsvp-message').value.trim();
                const msgBox = document.getElementById('rsvp-message-box');
                
                if (!window.saveRsvp) { msgBox.textContent = "Fitur database tidak aktif."; msgBox.classList.remove('hidden'); return; }

                msgBox.classList.remove('hidden', 'bg-red-100', 'text-red-800', 'bg-green-100', 'text-green-800');
                msgBox.textContent = 'Sedang mengirim konfirmasi...';
                
                const result = await window.saveRsvp(name, attendance, message);
                
                if (result.success) {
                    msgBox.classList.add('bg-green-100', 'text-green-800');
                    document.getElementById('rsvp-form').reset();
                } else {
                    msgBox.classList.add('bg-red-100', 'text-red-800');
                }
                msgBox.textContent = result.message;
            });
            
            // 6. Guestbook Form Submission (Uses global saveGuestbookEntry)
            document.getElementById('guestbook-form').addEventListener('submit', async (e) => {
                e.preventDefault();
                const name = document.getElementById('guestbook-name').value.trim();
                const message = document.getElementById('guestbook-message').value.trim();
                const msgBox = document.getElementById('guestbook-message-box');
                
                if (!window.saveGuestbookEntry) { msgBox.textContent = "Fitur database tidak aktif."; msgBox.classList.remove('hidden'); return; }

                msgBox.classList.remove('hidden', 'bg-red-100', 'text-red-800', 'bg-green-100', 'text-green-800');
                msgBox.textContent = 'Sedang mengirim ucapan...';

                const result = await window.saveGuestbookEntry(name, message);

                if (result.success) {
                    msgBox.classList.add('bg-green-100', 'text-green-800');
                    document.getElementById('guestbook-form').reset();
                } else {
                    msgBox.classList.add('bg-red-100', 'text-red-800');
                }
                msgBox.textContent = result.message;
            });

            // 7. Scroll Animation (Intersection Observer for smooth fade-in)
            function setupScrollAnimations() {
                const elements = document.querySelectorAll('.animate-fade-in:not(.is-visible)');

                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            // Apply a delay based on the element's CSS `delay-*` class if present
                            const delayClass = Array.from(entry.target.classList).find(cls => cls.startsWith('delay-'));
                            const delayMs = delayClass ? parseInt(delayClass.split('-')[1]) : 0;

                            setTimeout(() => {
                                entry.target.classList.add('is-visible');
                                observer.unobserve(entry.target); // Stop observing once visible
                            }, delayMs);
                        }
                    });
                }, {
                    root: null,
                    rootMargin: '0px',
                    threshold: 0.1 // Trigger when 10% of element is visible
                });

                elements.forEach(el => observer.observe(el));
            }

            // 8. Simple Parallax Effect (applied only to Hero BG)
            window.addEventListener('scroll', () => {
                const hero = document.getElementById('hero-section');
                // Ensure hero element exists and we are past the gatekeeper
                if (hero && !gatekeeper.classList.contains('hidden')) { 
                    const scrollY = window.scrollY;
                    // Move background image slightly slower than scroll (0.5 speed)
                    hero.style.backgroundPositionY = (scrollY * 0.5) + 'px';
                }
            });

        });
    </script>

</body>
</html>
