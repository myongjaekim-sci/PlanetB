# PlanetB
Trophic Level ID and Creating a sustainable Ecosystem
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planet B: Ecological Survey</title>
    <style>
        :root {
            --bg: #0b0e14;
            --card: #1e293b;
            --accent: #38bdf8;
            --producer: #4ade80;
            --primary: #fbbf24;
            --secondary: #f87171;
            --text: #f1f5f9;
            --error: #ef4444;
        }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        #game-container { width: 100%; max-width: 1100px; }

        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #334155; padding-bottom: 10px; }

        /* The Grid of Unknowns */
        #discovery-field {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
            gap: 10px;
            background: #111827;
            padding: 15px;
            border-radius: 12px;
            border: 2px dashed #334155;
            margin-bottom: 20px;
            min-height: 100px;
        }

        .organism {
            background: var(--card);
            border: 1px solid #475569;
            border-radius: 6px;
            padding: 10px;
            cursor: pointer;
            text-align: center;
            font-size: 0.85em;
            transition: all 0.2s;
            user-select: none;
        }

        .organism:hover { border-color: var(--accent); background: #2d3e50; }
        .organism.misplaced { border: 2px solid var(--error); animation: shake 0.5s; }

        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-4px); }
            75% { transform: translateX(4px); }
        }

        /* Sorting Zones */
        .zones {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 15px;
            margin-bottom: 30px;
        }

        .zone {
            background: #161e2e;
            border-radius: 12px;
            padding: 15px;
            min-height: 400px;
            border-top: 4px solid #ccc;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        #zone-producer { border-color: var(--producer); }
        #zone-primary { border-color: var(--primary); }
        #zone-secondary { border-color: var(--secondary); }

        .zone h3 { text-align: center; margin: 0 0 10px 0; font-size: 0.9em; text-transform: uppercase; letter-spacing: 1px; }

        /* Controls */
        #audit-bar {
            position: sticky;
            bottom: 20px;
            background: var(--card);
            padding: 15px 30px;
            border-radius: 50px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            display: flex;
            gap: 20px;
            align-items: center;
            border: 1px solid var(--accent);
        }

        button {
            padding: 10px 20px;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }

        #audit-btn { background: var(--accent); color: #0f172a; font-size: 1.1em; }
        #audit-btn:disabled { background: #475569; cursor: not-allowed; }

        /* Modal */
        #modal-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9);
            display: none; justify-content: center; align-items: center; z-index: 1000;
        }

        .modal {
            background: var(--card);
            padding: 30px;
            border-radius: 15px;
            max-width: 450px;
            width: 90%;
            text-align: center;
            border: 2px solid var(--accent);
        }

        .modal h2 { color: var(--accent); }
        .btn-grid { display: grid; grid-template-columns: 1fr; gap: 10px; margin-top: 20px; }
        .move-btn { color: #000; }
        .btn-p { background: var(--producer); }
        .btn-c1 { background: var(--primary); }
        .btn-c2 { background: var(--secondary); }
        .btn-close { background: #64748b; color: white; }

        #success-msg { color: var(--producer); font-weight: bold; display: none; }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <h1>PLANET B: ECOLOGICAL AUDIT</h1>
        <p>Classify all 36 lifeforms. No feedback will be given until you run the Audit.</p>
    </header>

    <div id="discovery-field"></div>

    <div class="zones">
        <div class="zone" id="zone-producer" data-type="producer"><h3>Producers</h3></div>
        <div class="zone" id="zone-primary" data-type="primary"><h3>Primary Consumers</h3></div>
        <div class="zone" id="zone-secondary" data-type="secondary"><h3>Secondary Consumers</h3></div>
    </div>

    <div id="audit-bar">
        <div>Progress: <span id="placed-count">0</span> / 36</div>
        <button id="audit-btn" onclick="runAudit()" disabled>RUN ECOSYSTEM AUDIT</button>
        <div id="success-msg">✓ ECOSYSTEM STABILIZED</div>
    </div>
</div>

<div id="modal-overlay">
    <div class="modal">
        <h2 id="m-name"></h2>
        <p id="m-desc"></p>
        <div class="btn-grid">
            <button class="move-btn btn-p" onclick="moveTo('producer')">Assign as Producer</button>
            <button class="move-btn btn-c1" onclick="moveTo('primary')">Assign as Primary Consumer</button>
            <button class="move-btn btn-c2" onclick="moveTo('secondary')">Assign as Secondary Consumer</button>
            <button class="btn-close" onclick="closeModal()">Close Observation</button>
        </div>
    </div>
</div>

<script>
    const data = [
        // PRODUCERS
        { id: 1, name: "Neon Fern", type: "producer", desc: "Absorbs radiation from the twin suns to create energy." },
        { id: 2, name: "Glow-Moss", type: "producer", desc: "Clings to rocks, converting mineral salts and starlight into sugars." },
        { id: 3, name: "Glass Grass", type: "producer", desc: "Transparent blades that use silica to maximize photosynthesis." },
        { id: 4, name: "Pulse-Bloom", type: "producer", desc: "A flower that releases oxygen when hit by lightning." },
        { id: 5, name: "Solar Lily", type: "producer", desc: "Floats on methane lakes, soaking up ultraviolet light." },
        { id: 6, name: "Aether-Vine", type: "producer", desc: "Suspended in mid-air, it pulls carbon directly from the clouds." },
        { id: 7, name: "Crystal Cacti", type: "producer", desc: "Stores energy in vibrant violet prismatic cells." },
        { id: 8, name: "Static-Shrub", type: "producer", desc: "Generates its own food using the planet's magnetic field." },
        { id: 9, name: "Void-Kelp", type: "producer", desc: "Deep-sea flora that survives on volcanic heat and chemical vents." },
        { id: 10, name: "Amber-Stalk", type: "producer", desc: "Produces sap through the process of 'stellar-synthesis'." },
        { id: 11, name: "Nebula-Orchid", type: "producer", desc: "Breathes in toxic gases and exhales pure energy spores." },
        { id: 12, name: "Frost-Petal", type: "producer", desc: "Found in ice caps, it turns frozen nitrogen into nutrients." },
        // PRIMARY
        { id: 13, name: "Hover-Slug", type: "primary", desc: "Slowly drifts between Neon Ferns, grazing on their glowing leaves." },
        { id: 14, name: "Silk-Beetle", type: "primary", desc: "Has specialized mandibles for crushing Solar Lily seeds." },
        { id: 15, name: "Mist-Rabbit", type: "primary", desc: "Feeds exclusively on the spores of Nebula Orchids." },
        { id: 16, name: "Shard-Snail", type: "primary", desc: "Eat the mineral-rich Glass Grass to build its tough shell." },
        { id: 17, name: "Prism-Moth", type: "primary", desc: "Drinks the high-energy nectar from Pulse-Blooms." },
        { id: 18, name: "Cloud-Grazer", type: "primary", desc: "A floating entity that nibbles on Aether-Vines high in the sky." },
        { id: 19, name: "Tundra-Cavy", type: "primary", desc: "Digs under the ice to find hidden Frost-Petal roots." },
        { id: 20, name: "Glitch-Goat", type: "primary", desc: "Eats Static-Shrubs, which causes its fur to spark with blue light." },
        { id: 21, name: "Methane-Turtle", type: "primary", desc: "A slow swimmer that grazes on Void-Kelp forests." },
        { id: 22, name: "Drift-Deer", type: "primary", desc: "Migrates across the plains eating Amber-Stalk seedlings." },
        { id: 23, name: "Glow-Cricket", type: "primary", desc: "Common pest that devours patches of Glow-Moss." },
        { id: 24, name: "Quartz-Borer", type: "primary", desc: "A worm-like creature that eats the inside of Crystal Cacti." },
        // SECONDARY
        { id: 25, name: "Phase-Panther", type: "secondary", desc: "Blends into shadows to hunt Hover-Slugs and Drift-Deer." },
        { id: 26, name: "Sky-Raptor", type: "secondary", desc: "Swoops down from the upper atmosphere to snatch Cloud-Grazers." },
        { id: 27, name: "Cobalt-Spider", type: "secondary", desc: "Spins webs made of liquid metal to catch Silk-Beetles." },
        { id: 28, name: "Void-Shark", type: "secondary", desc: "Patrols the dark methane lakes looking for Methane-Turtles." },
        { id: 29, name: "Static-Wolf", type: "secondary", desc: "Hunts in packs to take down the larger Glitch-Goats." },
        { id: 30, name: "Frost-Owl", type: "secondary", desc: "Uses thermal vision to find Mist-Rabbits hiding in the snow." },
        { id: 31, name: "Echo-Lizard", type: "secondary", desc: "Captures Glow-Crickets with a long, sticky tongue." },
        { id: 32, name: "Crag-Viper", type: "secondary", desc: "Lurks near crystal caves to ambush Quartz-Borers." },
        { id: 33, name: "Mirror-Hawk", type: "secondary", desc: "Highly intelligent bird that hunts Prism-Moths mid-flight." },
        { id: 34, name: "Plasma-Bear", type: "secondary", desc: "Smashes Shard-Snail shells with its powerful magnetic claws." },
        { id: 35, name: "Tundra-Fox", type: "secondary", desc: "A quick predator that thrives on a diet of Tundra-Cavies." },
        { id: 36, name: "Nova-Wasp", type: "secondary", desc: "Injects venom into herbivores, using them as hosts for larvae." }
    ];

    let currentOrgId = null;
    let placedCount = 0;
    const organismMap = {};

    function init() {
        const field = document.getElementById('discovery-field');
        const shuffled = [...data].sort(() => Math.random() - 0.5);
        
        shuffled.forEach(item => {
            const div = document.createElement('div');
            div.className = 'organism';
            div.id = `org-${item.id}`;
            div.innerHTML = `<b>UNKNOWN</b><br>ID: ${item.id}`;
            div.onclick = () => openModal(item.id);
            field.appendChild(div);
            organismMap[item.id] = { ...item, currentZone: 'field' };
        });
    }

    function openModal(id) {
        currentOrgId = id;
        const org = organismMap[id];
        document.getElementById('m-name').innerText = org.name;
        document.getElementById('m-desc').innerText = org.desc;
        document.getElementById('modal-overlay').style.display = 'flex';
    }

    function closeModal() {
        document.getElementById('modal-overlay').style.display = 'none';
    }

    function moveTo(zoneType) {
        const org = organismMap[currentOrgId];
        const el = document.getElementById(`org-${currentOrgId}`);
        
        // Remove 'misplaced' style whenever moved
        el.classList.remove('misplaced');
        
        if (org.currentZone === 'field') {
            placedCount++;
        }

        org.currentZone = zoneType;
        el.innerHTML = `<b>${org.name}</b>`;
        document.getElementById(`zone-${zoneType}`).appendChild(el);
        
        updateUI();
        closeModal();
    }

    function updateUI() {
        document.getElementById('placed-count').innerText = placedCount;
        document.getElementById('audit-btn').disabled = placedCount < 36;
    }

    function runAudit() {
        let allCorrect = true;
        let errors = 0;

        Object.values(organismMap).forEach(org => {
            const el = document.getElementById(`org-${org.id}`);
            if (org.currentZone !== org.type) {
                el.classList.add('misplaced');
                allCorrect = false;
                errors++;
            } else {
                el.classList.remove('misplaced');
            }
        });

        if (allCorrect) {
            document.getElementById('success-msg').style.display = 'block';
            document.getElementById('audit-btn').style.display = 'none';
            alert("SUCCESS! The ecosystem audit is clear. All trophic levels are balanced.");
        } else {
            alert(`AUDIT FAILED: ${errors} bio-signatures are misplaced. Re-examine the observations for the highlighted organisms.`);
        }
    }

    init();
</script>
</body>
</html>
