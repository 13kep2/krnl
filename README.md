<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>TAV5C // KERNEL PANIC</title>
    <link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&display=swap" rel="stylesheet"/>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        :root {
            --crimson: #ff0033;
            --bg: #020000;
            --glass: rgba(20, 0, 0, 0.55);
            --text: #e6d8c3;
        }
            Background Video
            #bg-video {
            position: fixed;
            top: 50%;
            left: 50%;
            min-width: 100%;
            min-height: 100%;
            width: auto;
            height: auto;
            transform: translate(-50%, -50%);
            z-index: -2;
            object-fit: cover;
        }
        body {
            background: radial-gradient(circle at 50% 20%, #120000, #020000 80%);
            color: var(--text);
            font-family: 'Space Mono', monospace;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 40px;
            overflow: hidden;
            position: relative;
        }

body::before {
            content: "";
            position: fixed;
            inset: 0;
            background: repeating-linear-gradient(
                0deg,
                rgba(255, 0, 51, .04) 0px,
                rgba(255, 0, 51, .04) 1px,
                transparent 1px,
                transparent 3px
            );
            pointer-events: none;
            z-index: -1;
        }

#discord-card {
            width: 460px;
            background: var(--glass);
            border: 1px solid rgba(255, 0, 51, .25);
            border-radius: 18px;
            backdrop-filter: blur(16px);
            box-shadow: 0 0 40px rgba(255, 0, 51, .15);
            animation: flicker 6s infinite;
            z-index: 10;
            position: relative;
        }

@keyframes flicker {
            0%, 100% { opacity: 1; }
            3% { opacity: .9; }
            6% { opacity: 1; }
        }

#dc-pill {
            display: flex;
            gap: 16px;
            padding: 20px;
            align-items: center;
        }

#dc-avatar {
            width: 72px;
            height: 72px;
            border-radius: 10px;
            border: 2px solid var(--crimson);
            object-fit: cover;
        }

#dc-name {
            font-size: 1.2rem;
            font-weight: 700;
        }

#dc-srow {
            display: flex;
            align-items: center;
            gap: 8px;
        }

#dc-status-txt {
            font-size: .9rem;
        }

#dc-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
        }

.online { background: #43b581; }
        .idle { background: #faa61a; }
        .dnd { background: #f04747; }
        .offline { background: #555; }

#dc-body {
            border-top: 1px solid rgba(255, 0, 51, .2);
            padding: 16px 20px;
        }

#dc-act-name {
            font-size: .85rem;
            opacity: .6;
        }

#dc-act-detail {
            font-size: .8rem;
            line-height: 1.6;
        }

typing {
            display: flex;
            gap: 6px;
            align-items: center;
            height: 16px;
        }

.typing span {
            width: 6px;
            height: 6px;
            border-radius: 50%;
            background: var(--crimson);
            opacity: .4;
            animation: typing 1.2s infinite;
        }
        .typing span:nth-child(2) { animation-delay: .2s; }
        .typing span:nth-child(3) { animation-delay: .4s; }

@keyframes typing {
            0% { transform: translateY(0); opacity: .3; }
            50% { transform: translateY(-4px); opacity: 1; }
            100% { transform: translateY(0); opacity: .3; }
        }
    </style>
</head>
<body>
<!-- Background Video (notbg.mp4) - wallpaper style, no sound -->
    
<video id="bg-video" autoplay loop muted playsinline>
        <source src="notbg.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>

<div id="discord-card">
        <div id="dc-pill">
            <img id="dc-avatar">
            <div>
                <div id="dc-name">ikuyo</div>
                <div id="dc-srow">
                    <div id="dc-dot" class="online"></div>
                    <span id="dc-status-txt">online</span>
                </div>
            </div>
        </div>
        <div id="dc-body">
            <div id="dc-act-name">Status</div>
            <div id="dc-act-detail">loading...</div>
        </div>
    </div>
<!-- Background Music (main.mp3) - loop + autoplay -->
    <audio id="bg-music" autoplay loop>
        <source src="main.mp3" type="audio/mpeg">
        Your browser does not support the audio element.
    </audio>

<script>
        const CFG = {
            discordId: "899213421587865620"
        };
        let avatarSet = false;
        let cachedData = null;

        function showTyping() {
            document.getElementById("dc-act-name").textContent = "Typing...";
            document.getElementById("dc-act-detail").innerHTML =
                `<div class="typing"><span></span><span></span><span></span></div>`;
        }

async function fetchPresence() {
            try {
                const res = await fetch(`https://api.lanyard.rest/v1/users/${CFG.discordId}`);
                const json = await res.json();
                if (!json.success) return;
                cachedData = json.data;
                const u = cachedData.discord_user;
                if (!avatarSet) {
                    document.getElementById("dc-avatar").src =
                        `https://cdn.discordapp.com/avatars/${u.id}/${u.avatar}.png?size=256`;
                    avatarSet = true;
                }
            } catch (e) {
                showTyping();
            }
        }

function updateUI() {
            if (!cachedData) return;
            const d = cachedData;
            const status = d.discord_status || "offline";
            const dot = document.getElementById("dc-dot");
            dot.className = "";
            dot.classList.add(status);
            document.getElementById("dc-status-txt").textContent =
                status === "dnd" ? "do not disturb" : status;

const custom = d.activities?.find(a => a.type === 4);
            if (custom) {
                document.getElementById("dc-act-name").textContent = "Custom Status";
                document.getElementById("dc-act-detail").textContent =
                    custom.state || "No status";
} else if (d.activities?.length) {
                const act = d.activities[0];
                document.getElementById("dc-act-name").textContent = act.name;
                document.getElementById("dc-act-detail").textContent =
                    act.details || act.state || "";
            } else {
                showTyping();
            }
        }

// Initial load
        fetchPresence();
        updateUI();

// Refresh intervals
        setInterval(fetchPresence, 25000);
        setInterval(updateUI, 1500);

// Optional: Restart audio if it gets blocked by browser
        const music = document.getElementById("bg-music");
        music.volume = 0.4; // Adjust volume if needed (0.0 - 1.0)
</script>
</body>
</html>
