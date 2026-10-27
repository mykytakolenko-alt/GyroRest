## GyroRest is the orientation based Inactivity Alerts for Iphone/Android phones.

✨Have you ever put your phone down and immediately received a push notification?

Try similar experience and easily implement this feature into your application by copying this entire <script> block into your project.

<img width="972" height="732" alt="image" src="https://github.com/user-attachments/assets/c0ca8407-5529-4653-8fce-9c3c9689be2e" />



## ✨ What does it do?

-Plays sound and sends notification after 10 seconds of no movement based on gyro sensors of the phone.

-If you still don't pick it up, it nags you again at 80s, 160s, and 240s.

The moment you touch the phone - everything resets.

Try it live: https://gyrorest.netlify.app/

## How to implement this feature into your project: 
'''js
<script>
        /*
         * INTEGRATION GUIDE:
         * 1. Copy this entire <script> block into your project.
         * 2. Add HTML elements with IDs: 'status', 'startBtn', 'termsModal', 'closeTerms', 'acceptBtn', 'termsLink'.
         * 3. Copy CSS classes: .status (with :empty rule), .active, .idle, .sleep, .error, @keyframes pulse.
         * 4. Serve over HTTPS (required for DeviceOrientation API on mobile).
         * 5. Adjust CONFIG values below to customize timing and sensitivity.
         */

        // CONFIG: Customize alert timing and motion detection
        const CONFIG = {
            alerts: [10, 80, 160, 240],  // Seconds of stillness before each alert (add/remove as needed)
            threshold: 3,                  // Max rotation (degrees) to count as "still"
            tickMs: 250                    // Timer interval for checking elapsed time
        };

        // State management
        const state = { watching: false, termsAccepted: false, audioCtx: null, timer: null, latest: null, anchor: null, stillSince: 0, fired: [] };
        const els = { status: document.getElementById('status'), startBtn: document.getElementById('startBtn'), modal: document.getElementById('termsModal') };

        // Update status bar (auto-hides when empty via CSS)
        const setStatus = (text, cls) => { 
            els.status.textContent = text; 
            els.status.className = 'status ' + (cls || ''); 
        };

        // Audio: Play C-E-G major chord chime
        const playChime = () => {
            if (!state.audioCtx) state.audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            if (state.audioCtx.state === 'suspended') state.audioCtx.resume();
            [523.25, 659.25, 783.99].forEach((freq, i) => {
                const osc = state.audioCtx.createOscillator(), gain = state.audioCtx.createGain();
                const t = state.audioCtx.currentTime + i * 0.05;
                osc.frequency.value = freq;
                gain.gain.setValueAtTime(0, t);
                gain.gain.linearRampToValueAtTime(0.2, t + 0.05);
                gain.gain.exponentialRampToValueAtTime(0.001, t + 0.8);
                osc.connect(gain); gain.connect(state.audioCtx.destination);
                osc.start(t); osc.stop(t + 0.8);
            });
        };

        // Calculate angle difference (handles 360° wrap-around)
        const angleDiff = (a, b) => Math.abs(((a - b + 540) % 360) - 180);
        const deviation = (p, q) => angleDiff(p.a, q.a) + angleDiff(p.b, q.b) + angleDiff(p.g, q.g);

        // Sensor callback: Store orientation and initialize baseline on first reading
        const onOrientation = (e) => {
            if (e.alpha === null) return;
            state.latest = { a: e.alpha || 0, b: e.beta || 0, g: e.gamma || 0 };
            if (!state.anchor) { state.anchor = state.latest; state.stillSince = Date.now(); state.fired = CONFIG.alerts.map(() => false); }
        };

        // Timer tick: Check if phone is still and trigger alerts
        const tick = () => {
            if (!state.watching || !state.latest) return setStatus('👀 Waiting for sensor data…', 'active');
            if (deviation(state.latest, state.anchor) > CONFIG.threshold) {
                state.anchor = state.latest; state.stillSince = Date.now(); state.fired = CONFIG.alerts.map(() => false);
                return setStatus(' Phone is in motion', 'active');
            }
            const elapsed = (Date.now() - state.stillSince) / 1000;
            setStatus(`📱 Phone is still for ${elapsed.toFixed(0)}s…`, 'idle');
            CONFIG.alerts.forEach((sec, i) => {
                if (!state.fired[i] && elapsed >= sec) {
                    state.fired[i] = true;
                    setStatus('💤 Phone is resting. Alert triggered!', 'sleep');
                    playChime();
                    if (navigator.vibrate) navigator.vibrate([150, 100, 150]);
                    if (Notification.permission === 'granted') new Notification('Hi 👋', { body: "It's been a while!", tag: 'gyrorest' });
                }
            });
        };

        // Start/stop monitoring with permission handling
        const toggleMonitoring = async () => {
            if (state.watching) {
                window.removeEventListener('deviceorientation', onOrientation);
                clearInterval(state.timer);
                state.watching = false;
                els.startBtn.textContent = '▶ Start Monitoring';
                return setStatus('');
            }
            if (!state.termsAccepted) return els.modal.style.display = 'flex';
            if (typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission === 'function') {
                if (await DeviceOrientationEvent.requestPermission() !== 'granted') return setStatus('Motion sensor permission denied.', 'error');
            }
            if ('Notification' in window && Notification.permission === 'default') await Notification.requestPermission().catch(() => {});
            
            playChime();
            window.addEventListener('deviceorientation', onOrientation);
            state.timer = setInterval(tick, CONFIG.tickMs);
            state.watching = true;
            els.startBtn.textContent = ' Stop Monitoring';
            setStatus('👀 Listening to sensors…', 'active');
        };

        // Event listeners
        els.startBtn.addEventListener('click', toggleMonitoring);
        document.getElementById('acceptBtn').addEventListener('click', () => { state.termsAccepted = true; els.modal.style.display = 'none'; if (!state.watching) toggleMonitoring(); });
        document.getElementById('closeTerms').addEventListener('click', () => els.modal.style.display = 'none');
        document.getElementById('termsLink').addEventListener('click', e => { e.preventDefault(); els.modal.style.display = 'flex'; });
        els.modal.addEventListener('click', e => { if (e.target === els.modal) els.modal.style.display = 'none'; });
        document.addEventListener('keydown', e => { if (e.key === 'Escape') els.modal.style.display = 'none'; });
        
        // Pause monitoring when tab is hidden to save battery
        document.addEventListener('visibilitychange', () => {
            if (!state.watching) return;
            if (document.hidden) { window.removeEventListener('deviceorientation', onOrientation); clearInterval(state.timer); }
            else { window.addEventListener('deviceorientation', onOrientation); state.timer = setInterval(tick, CONFIG.tickMs); state.anchor = null; setStatus(' Listening to sensors…', 'active'); }
        });
    </script>
