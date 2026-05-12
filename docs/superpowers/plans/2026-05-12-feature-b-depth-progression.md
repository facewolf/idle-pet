# Feature B: Depth & Progression System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add pet leveling/XP, skill trees per pet, evolution stage visuals, and global mastery tracks.

**Architecture:** Extend the existing `state` object with new properties for `petXP`, `petLevels`, `petSkills`, `masteryLevels`. Add XP gain from click/sell/pet actions. Implement 3-branch skill trees. Add visual evolution suffix in collection display. Add mastery panel with 4 categories.

**Tech Stack:** Single HTML file, vanilla JavaScript, localStorage persistence

---

## File Structure

**Modify:**
- `index.html` - All changes in this single file

---

## Implementation Tasks

### Task 1: Pet Leveling System (Foundation)

**Files:**
- Modify: `index.html:616-643` (state initialization)
- Modify: `index.html:700-702` (clickEgg function)
- Modify: `index.html:741-773` (sellEggs function)
- Modify: `index.html:819-848` (spawnPet function)
- Modify: `index.html:1570-1601` (gameTick)
- Modify: `index.html:1604-1643` (updateUI)
- Modify: `index.html:1645-1659` (renderCollection)

- [ ] **Step 1: Update state initialization**

Add to state object at line ~617:
```javascript
petXP: {},        // { "🐶": 0 }
petLevels: {},    // { "🐶": 1 }
```

- [ ] **Step 2: Add XP gain function**

Add before `formatNumber` function (~line 645):
```javascript
function getXPForLevel(level) {
    return Math.floor(100 * Math.pow(1.5, level - 1));
}

function addPetXP(pet, amount) {
    if (!state.petXP[pet]) state.petXP[pet] = 0;
    if (!state.petLevels[pet]) state.petLevels[pet] = 1;

    state.petXP[pet] += amount;
    const xpNeeded = getXPForLevel(state.petLevels[pet]);

    while (state.petXP[pet] >= xpNeeded && state.petLevels[pet] < 100) {
        state.petXP[pet] -= xpNeeded;
        state.petLevels[pet]++;
        showToast('⭐ ' + pet + ' leveled up to ' + state.petLevels[pet] + '!');
    }
}
```

- [ ] **Step 3: Add XP gain to clickEgg**

Modify clickEgg function (~line 700):
```javascript
function clickEgg() {
    const eggType = getEggType();
    const amount = getEggAmount(eggType) * (1 + state.eggValueLevel * 0.1);
    state.eggs += amount;
    state.stats.totalClicks++;

    // Grant XP to random pet if player has any
    if (state.pets.length > 0) {
        const randomPet = state.pets[Math.floor(Math.random() * state.pets.length)];
        addPetXP(randomPet, 1);
    }

    showClickEffect(Math.floor(amount), eggType);
    // ... rest of existing code
}
```

- [ ] **Step 4: Add XP gain to sellEggs**

Add XP grant in sellEggs function (~line 741) before `checkAchievements()`:
```javascript
// Grant XP based on coins earned
state.pets.forEach(pet => {
    addPetXP(pet, Math.floor(coinsEarned * 0.1));
});
```

- [ ] **Step 5: Add level display in collection**

Modify renderCollection (~line 1645) to show level:
```javascript
<div onclick="openTrainingModal('${pet}')" ...>
    <span class="text-3xl">${pet}${getEvolutionSuffix(evoLevel)}</span>
    ${acc ? '<span class="text-lg absolute -top-1 -right-1">' + acc + '</span>' : ''}
    ${state.petLevels[pet] ? '<span class="text-xs bg-blue-600 rounded-full px-1">Lv' + state.petLevels[pet] + '</span>' : ''}
    ${state.petCounts[pet] > 1 ? '<span class="text-xs bg-red-500 rounded-full px-1">x${state.petCounts[pet]}</span>' : ''}
    ${Object.values(state.petTraining[pet] || {}).some(v => v > 0) ? '<span class="text-xs text-yellow-400">★</span>' : ''}
</div>
```

- [ ] **Step 6: Add Pet Level modal for details**

Add modal HTML before `<!-- Pet Training Modal -->` (~line 190):
```html
<!-- Pet Level Modal -->
<div id="levelModal" class="fixed inset-0 bg-black/80 flex items-center justify-center z-50 hidden">
    <div class="bg-card/90 backdrop-blur rounded-2xl p-8 max-w-md w-full mx-4 max-h-[80vh] overflow-y-auto">
        <h2 class="text-2xl font-bold mb-4">⭐ Pet Level Details</h2>
        <div id="levelPetList" class="space-y-3"></div>
        <button onclick="closeLevelModal()" class="mt-4 bg-purple-600 px-6 py-2 rounded-lg w-full">Tutup</button>
    </div>
</div>
```

Add functions (~line 940):
```javascript
function openLevelModal() {
    const list = document.getElementById('levelPetList');
    if (state.pets.length === 0) {
        list.innerHTML = '<p class="text-gray-500 text-center">Belum punya pet!</p>';
    } else {
        list.innerHTML = state.pets.map(pet => {
            const level = state.petLevels[pet] || 1;
            const xp = state.petXP[pet] || 0;
            const xpNeeded = getXPForLevel(level);
            const pct = Math.min(100, Math.floor((xp / xpNeeded) * 100));
            return `
                <div class="bg-blue-900/30 rounded-lg p-3 flex items-center gap-3">
                    <span class="text-3xl">${pet}${getEvolutionSuffix(state.petEvolution[pet] || 0)}</span>
                    <div class="flex-1">
                        <div class="flex justify-between text-sm mb-1">
                            <span>Level ${level}</span>
                            <span>${xp}/${xpNeeded} XP</span>
                        </div>
                        <div class="quest-progress">
                            <div class="quest-progress-bar" style="width:${pct}%"></div>
                        </div>
                    </div>
                </div>
            `;
        }).join('');
    }
    document.getElementById('levelModal').classList.remove('hidden');
}

function closeLevelModal() {
    document.getElementById('levelModal').classList.add('hidden');
}
```

- [ ] **Step 7: Add button to header to open level modal**

Add to header buttons (~line 263):
```html
<button onclick="openLevelModal()" class="text-sm bg-card/50 px-3 py-1 rounded-full hover:bg-blue-600 transition-colors">⭐ Levels</button>
```

- [ ] **Step 8: Update loadGame to initialize new state properties**

Add to loadGame (~line 1680):
```javascript
state.petXP = state.petXP || {};
state.petLevels = state.petLevels || {};
```

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add pet leveling and XP system"
```

---

### Task 2: Skill Trees per Pet

**Files:**
- Modify: `index.html:616-643` (state)
- Modify: `index.html:940-953` (openTrainingModal)

- [ ] **Step 1: Add petSkills to state initialization**

Add to state object:
```javascript
petSkills: {},    // { "🐶": { offense: 0, defense: 0, utility: 0 } }
```

- [ ] **Step 2: Add skill tree constants**

Add after ACCESSORIES (~line 598):
```javascript
const SKILL_BRANCHES = {
    offense: {
        name: '⚔️ Offense',
        skills: [
            { id: 'clickPower', name: 'Click Power', desc: '+5% click power per level', icon: '👊', cost: 100 },
            { id: 'eggValue', name: 'Egg Value', desc: '+5% egg value per level', icon: '🥚', cost: 120 },
            { id: 'critChance', name: 'Critical', desc: '+1% crit chance per level', icon: '💥', cost: 150 }
        ]
    },
    defense: {
        name: '🛡️ Defense',
        skills: [
            { id: 'autoSellEff', name: 'Auto-Sell', desc: '+5% auto-sell efficiency', icon: '🤖', cost: 100 },
            { id: 'luck', name: 'Luck', desc: '+2% luck per level', icon: '🍀', cost: 130 },
            { id: 'badLuckProt', name: 'Bad Luck Prot', desc: '-1% bad luck chance', icon: '🧲', cost: 140 }
        ]
    },
    utility: {
        name: '🔧 Utility',
        skills: [
            { id: 'spawnRate', name: 'Spawn Rate', desc: '+3% pet spawn rate', icon: '🐾', cost: 110 },
            { id: 'offlineEff', name: 'Offline Eff', desc: '+5% offline efficiency', icon: '🌙', cost: 100 },
            { id: 'dailyBonus', name: 'Daily Bonus', desc: '+5% daily reward', icon: '🎁', cost: 130 }
        ]
    }
};
```

- [ ] **Step 3: Initialize pet skills when pet is spawned**

Modify spawnPet (~line 840):
```javascript
if (!state.petTraining[newPet]) state.petTraining[newPet] = { speed: 0, luck: 0, value: 0 };
if (!state.petSkills[newPet]) state.petSkills[newPet] = { offense: 0, defense: 0, utility: 0 };
```

- [ ] **Step 4: Update openTrainingModal to include skill tree tab**

Replace the training modal content with tabbed interface:
```javascript
function openTrainingModal(pet) {
    selectedPetForTraining = pet;
    const info = document.getElementById('trainingPetInfo');
    const stats = document.getElementById('trainingStats');
    const t = state.petTraining[pet] || { speed: 0, luck: 0, value: 0 };
    const skills = state.petSkills[pet] || { offense: 0, defense: 0, utility: 0 };

    info.innerHTML = '<span class="text-5xl">' + pet + '</span><p class="text-sm text-gray-400">Pet Training & Skill Tree</p>';

    // Tab buttons
    const tabs = `
        <div class="flex gap-2 mb-4">
            <button onclick="showTrainingTab('stats')" class="flex-1 bg-blue-600 px-3 py-2 rounded-lg text-sm">📊 Stats</button>
            <button onclick="showTrainingTab('skills')" class="flex-1 bg-orange-600 px-3 py-2 rounded-lg text-sm">⚔️ Skills</button>
        </div>
    `;

    // Stats content
    const statsContent = `
        <div class="flex items-center justify-between bg-blue-900/30 p-3 rounded-lg">
            <div><span class="font-bold">⚡ Speed</span><br><span class="text-xs text-gray-400">Spawn lebih cepat</span></div>
            <div class="text-right">
                <span class="text-xl font-bold">${t.speed}/10</span>
                <button onclick="trainStat('${pet}', 'speed')" class="block mt-1 bg-blue-600 hover:bg-blue-500 px-3 py-1 rounded text-xs" ${t.speed >= 10 ? 'disabled' : ''}>
                    🪙 ${50 * (t.speed + 1)}
                </button>
            </div>
        </div>
        <div class="flex items-center justify-between bg-pink-900/30 p-3 rounded-lg">
            <div><span class="font-bold">🍀 Luck</span><br><span class="text-xs text-gray-400">+2% rare per level</span></div>
            <div class="text-right">
                <span class="text-xl font-bold">${t.luck}/10</span>
                <button onclick="trainStat('${pet}', 'luck')" class="block mt-1 bg-pink-600 hover:bg-pink-500 px-3 py-1 rounded text-xs" ${t.luck >= 10 ? 'disabled' : ''}>
                    🪙 ${50 * (t.luck + 1)}
                </button>
            </div>
        </div>
        <div class="flex items-center justify-between bg-green-900/30 p-3 rounded-lg">
            <div><span class="font-bold">💰 Value</span><br><span class="text-xs text-gray-400">+10% sell value</span></div>
            <div class="text-right">
                <span class="text-xl font-bold">${t.value}/10</span>
                <button onclick="trainStat('${pet}', 'value')" class="block mt-1 bg-green-600 hover:bg-green-500 px-3 py-1 rounded text-xs" ${t.value >= 10 ? 'disabled' : ''}>
                    🪙 ${50 * (t.value + 1)}
                </button>
            </div>
        </div>
    `;

    // Skills content
    const skillsContent = `
        <div class="mb-3">
            <h4 class="font-bold text-orange-400 mb-2">⚔️ Offense Branch</h4>
            ${SKILL_BRANCHES.offense.skills.map(skill => {
                const level = skill.id === 'clickPower' ? skills.offense : skill.id === 'eggValue' ? skills.offense : skills.offense;
                const cost = skill.cost * (level + 1);
                return `
                    <div class="flex items-center justify-between bg-orange-900/20 p-2 rounded-lg mb-2">
                        <div>
                            <span class="text-lg">${skill.icon}</span>
                            <span class="font-semibold text-sm">${skill.name}</span>
                            <p class="text-xs text-gray-400">${skill.desc}</p>
                        </div>
                        <div class="text-right">
                            <span class="text-sm">Lv ${level}/3</span>
                            <button onclick="upgradePetSkill('${pet}', 'offense', '${skill.id}')" class="block mt-1 bg-orange-600 hover:bg-orange-500 px-2 py-1 rounded text-xs" ${level >= 3 || state.coins < cost ? 'disabled' : ''}>
                                🪙 ${cost}
                            </button>
                        </div>
                    </div>
                `;
            }).join('')}
        </div>
        <div class="mb-3">
            <h4 class="font-bold text-blue-400 mb-2">🛡️ Defense Branch</h4>
            ${SKILL_BRANCHES.defense.skills.map(skill => {
                const level = skill.id === 'autoSellEff' ? skills.defense : skill.id === 'luck' ? skills.defense : skills.defense;
                const cost = skill.cost * (level + 1);
                return `
                    <div class="flex items-center justify-between bg-blue-900/20 p-2 rounded-lg mb-2">
                        <div>
                            <span class="text-lg">${skill.icon}</span>
                            <span class="font-semibold text-sm">${skill.name}</span>
                            <p class="text-xs text-gray-400">${skill.desc}</p>
                        </div>
                        <div class="text-right">
                            <span class="text-sm">Lv ${level}/3</span>
                            <button onclick="upgradePetSkill('${pet}', 'defense', '${skill.id}')" class="block mt-1 bg-blue-600 hover:bg-blue-500 px-2 py-1 rounded text-xs" ${level >= 3 || state.coins < cost ? 'disabled' : ''}>
                                🪙 ${cost}
                            </button>
                        </div>
                    </div>
                `;
            }).join('')}
        </div>
        <div class="mb-3">
            <h4 class="font-bold text-green-400 mb-2">🔧 Utility Branch</h4>
            ${SKILL_BRANCHES.utility.skills.map(skill => {
                const level = skill.id === 'spawnRate' ? skills.utility : skill.id === 'offlineEff' ? skills.utility : skills.utility;
                const cost = skill.cost * (level + 1);
                return `
                    <div class="flex items-center justify-between bg-green-900/20 p-2 rounded-lg mb-2">
                        <div>
                            <span class="text-lg">${skill.icon}</span>
                            <span class="font-semibold text-sm">${skill.name}</span>
                            <p class="text-xs text-gray-400">${skill.desc}</p>
                        </div>
                        <div class="text-right">
                            <span class="text-sm">Lv ${level}/3</span>
                            <button onclick="upgradePetSkill('${pet}', 'utility', '${skill.id}')" class="block mt-1 bg-green-600 hover:bg-green-500 px-2 py-1 rounded text-xs" ${level >= 3 || state.coins < cost ? 'disabled' : ''}>
                                🪙 ${cost}
                            </button>
                        </div>
                    </div>
                `;
            }).join('')}
        </div>
    `;

    stats.innerHTML = tabs + '<div id="trainingTabContent"></div>';
    window.currentTrainingTab = 'stats';

    // Show stats by default
    document.getElementById('trainingTabContent').innerHTML = statsContent;

    document.getElementById('trainingModal').classList.remove('hidden');
}

function showTrainingTab(tab) {
    const content = document.getElementById('trainingTabContent');
    if (tab === 'stats') {
        // Insert statsContent here
    } else {
        // Insert skillsContent here
    }
}
```

Actually, let me simplify the implementation to avoid the complex tab switching:

- [ ] **Step 4: Simplify - Add skill tree below training stats in same modal**

Modify openTrainingModal to add skills section after stats:

```javascript
// After the 3 stat blocks, add skill section:
const skills = state.petSkills[pet] || { offense: 0, defense: 0, utility: 0 };
const skillsContent = `
    <div class="mt-4 pt-4 border-t border-gray-600">
        <h4 class="font-bold mb-3">⚔️ Skill Tree</h4>
        <div class="grid grid-cols-3 gap-2">
            ${['offense', 'defense', 'utility'].map(branch => {
                const branchData = SKILL_BRANCHES[branch];
                const level = skills[branch] || 0;
                const cost = 200 * (level + 1);
                return `
                    <div class="bg-${branch === 'offense' ? 'orange' : branch === 'defense' ? 'blue' : 'green'}-900/30 rounded-lg p-2 text-center">
                        <p class="text-sm font-bold">${branchData.name.split(' ')[1]}</p>
                        <p class="text-xs text-gray-400">Lv ${level}/3</p>
                        <button onclick="upgradePetSkillBranch('${pet}', '${branch}')" class="mt-1 bg-${branch === 'offense' ? 'orange' : branch === 'defense' ? 'blue' : 'green'}-600 hover:${branch === 'offense' ? 'orange' : branch === 'defense' ? 'blue' : 'green'}-500 px-2 py-1 rounded text-xs" ${level >= 3 || state.coins < cost ? 'disabled' : ''}>
                            🪙 ${cost}
                        </button>
                    </div>
                `;
            }).join('')}
        </div>
    </div>
`;
stats.innerHTML = statsContent + skillsContent;
```

- [ ] **Step 5: Add upgradePetSkillBranch function**

Add after closeTrainingModal (~line 940):
```javascript
function upgradePetSkillBranch(pet, branch) {
    const skills = state.petSkills[pet] || { offense: 0, defense: 0, utility: 0 };
    const level = skills[branch] || 0;
    const cost = 200 * (level + 1);

    if (level >= 3 || state.coins < cost) return;

    state.coins -= cost;
    skills[branch] = level + 1;
    state.petSkills[pet] = skills;

    showToast('⚔️ ' + pet + ' ' + SKILL_BRANCHES[branch].name + ' skill upgraded to Lv' + (level + 1) + '!');
    openTrainingModal(pet);
    updateUI();
}
```

- [ ] **Step 6: Update loadGame to initialize petSkills**

Add to loadGame:
```javascript
state.petSkills = state.petSkills || {};
```

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add skill trees per pet with 3 branches"
```

---

### Task 3: Evolution Stage Visual Updates

**Files:**
- Modify: `index.html:1645-1659` (renderCollection)
- Modify: `index.html:1056-1064` (updateAccessoryApplySection)

- [ ] **Step 1: Enhance evolution visual display**

Current evolution uses text suffix (+1, +2, +3). Let's enhance with background glow and border styles.

Modify getEvolutionBorder (~line 963):
```javascript
function getEvolutionBorder(level) {
    if (level === 0) return 'border-purple-500/30';
    if (level === 1) return 'border-green-500/50';
    if (level === 2) return 'border-purple-500/50 shadow-lg shadow-green-500/30';
    return 'border-yellow-500/50 shadow-lg shadow-yellow-500/50 animate-pulse';
}
```

- [ ] **Step 2: Add evolution progress indicator in collection**

Modify renderCollection to show evolution progress bar:
```javascript
<div onclick="openTrainingModal('${pet}')" ...>
    <div class="relative">
        <span class="text-3xl">${pet}${getEvolutionSuffix(evoLevel)}</span>
        ${evoLevel > 0 && evoLevel < 3 ? `
            <div class="absolute -bottom-1 left-1/2 -translate-x-1/2 w-full">
                <div class="bg-gray-700 rounded-full h-1">
                    <div class="bg-gradient-to-r from-green-400 to-yellow-400 h-1 rounded-full" style="width:${(evoLevel/3)*100}%"></div>
                </div>
            </div>
        ` : ''}
    </div>
    ...
</div>
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: enhance evolution visual with glow and progress bars"
```

---

### Task 4: Mastery Tracks (Global)

**Files:**
- Modify: `index.html:616-643` (state)
- Modify: `index.html:190-254` (add modal HTML)
- Add: UI elements in header

- [ ] **Step 1: Add mastery state**

Add to state:
```javascript
masteryLevels: { egg: 0, coin: 0, pet: 0, event: 0 },
masteryXP: { egg: 0, coin: 0, pet: 0, event: 0 },
```

- [ ] **Step 2: Add mastery constants**

Add after CHEST_REWARDS (~line 614):
```javascript
const MASTERY_TRACKS = {
    egg: { name: '🥚 Egg Mastery', desc: 'Master egg collection', color: 'purple', xpPerLevel: [100, 150, 200, 250, 300] },
    coin: { name: '🪙 Coin Mastery', desc: 'Master coin earning', color: 'yellow', xpPerLevel: [100, 150, 200, 250, 300] },
    pet: { name: '🐾 Pet Mastery', desc: 'Master pet collection', color: 'green', xpPerLevel: [100, 150, 200, 250, 300] },
    event: { name: '🎉 Event Mastery', desc: 'Master events', color: 'pink', xpPerLevel: [100, 150, 200, 250, 300] }
};
```

- [ ] **Step 3: Add mastery XP gain functions**

Add before formatNumber (~line 645):
```javascript
function getMasteryXPNeeded(category, level) {
    const base = MASTERY_TRACKS[category].xpPerLevel[Math.min(level, 4)];
    return Math.floor(base * Math.pow(1.5, level));
}

function addMasteryXP(category, amount) {
    if (!state.masteryXP[category]) state.masteryXP[category] = 0;
    if (!state.masteryLevels[category]) state.masteryLevels[category] = 0;

    state.masteryXP[category] += amount;
    const xpNeeded = getMasteryXPNeeded(category, state.masteryLevels[category]);

    while (state.masteryXP[category] >= xpNeeded && state.masteryLevels[category] < 50) {
        state.masteryXP[category] -= xpNeeded;
        state.masteryLevels[category]++;
        showToast('🏆 ' + MASTERY_TRACKS[category].name + ' - Level ' + state.masteryLevels[category] + '!');
        // Apply mastery bonus immediately
        applyMasteryBonus(category);
    }
}

function applyMasteryBonus(category) {
    const level = state.masteryLevels[category];
    // Different bonuses at each level milestone (5, 10, 15, etc.)
    if (level % 5 === 0) {
        switch(category) {
            case 'egg':
                state.eggValueBonus += 5;
                showToast('🥚 Egg Mastery bonus: +5% egg value!');
                break;
            case 'coin':
                // Bonus applied in coin earning functions
                break;
            case 'pet':
                // Bonus applied in pet spawn
                break;
            case 'event':
                // Bonus applied during events
                break;
        }
    }
}

function getMasteryBonus(category) {
    const level = state.masteryLevels[category] || 0;
    if (category === 'egg') return level * 2; // +2% egg value per level
    if (category === 'coin') return level * 1; // +1% coin value per level
    if (category === 'pet') return level * 1; // +1% spawn rate per level
    if (category === 'event') return level * 2; // +2% event bonus per level
    return 0;
}
```

- [ ] **Step 4: Add XP gain calls**

In clickEgg: `addMasteryXP('egg', 1);`
In sellEggs: `addMasteryXP('coin', Math.floor(coinsEarned * 0.2));`
In spawnPet: `addMasteryXP('pet', 5);`
In toggleEvent activation: `addMasteryXP('event', 50);`

- [ ] **Step 5: Add Mastery modal HTML**

Add modal before closing tag of other modals (~line 254):
```html
<!-- Mastery Modal -->
<div id="masteryModal" class="fixed inset-0 bg-black/80 flex items-center justify-center z-50 hidden">
    <div class="bg-card/90 backdrop-blur rounded-2xl p-8 max-w-md w-full mx-4 max-h-[80vh] overflow-y-auto">
        <h2 class="text-2xl font-bold mb-4">🏆 Mastery Tracks</h2>
        <p class="text-gray-400 text-sm mb-4">Masteries persist through prestige resets!</p>
        <div id="masteryList" class="space-y-3"></div>
        <button onclick="closeMasteryModal()" class="mt-4 bg-purple-600 px-6 py-2 rounded-lg w-full">Tutup</button>
    </div>
</div>
```

- [ ] **Step 6: Add mastery functions**

Add after closeAchModal (~line 1568):
```javascript
function openMasteryModal() {
    const list = document.getElementById('masteryList');
    list.innerHTML = Object.entries(MASTERY_TRACKS).map(([key, track]) => {
        const level = state.masteryLevels[key] || 0;
        const xp = state.masteryXP[key] || 0;
        const xpNeeded = getMasteryXPNeeded(key, level);
        const pct = Math.min(100, Math.floor((xp / xpNeeded) * 100));
        const bonus = getMasteryBonus(key);

        return `
            <div class="bg-${track.color}-900/30 rounded-xl p-4 border border-${track.color}-500/30">
                <div class="flex items-center gap-3 mb-2">
                    <span class="text-3xl">${track.name.split(' ')[0]}</span>
                    <div class="flex-1">
                        <p class="font-bold">${track.name.split(' ')[1]} ${track.name.split(' ')[2] || ''}</p>
                        <p class="text-xs text-gray-400">${track.desc}</p>
                    </div>
                    <span class="text-2xl font-bold text-${track.color}-300">Lv ${level}</span>
                </div>
                <div class="quest-progress mb-2">
                    <div class="quest-progress-bar" style="width:${pct}%"></div>
                </div>
                <div class="flex justify-between text-xs">
                    <span>${xp}/${xpNeeded} XP</span>
                    <span class="text-${track.color}-400">+${bonus}% bonus</span>
                </div>
            </div>
        `;
    }).join('');
    document.getElementById('masteryModal').classList.remove('hidden');
}

function closeMasteryModal() {
    document.getElementById('masteryModal').classList.add('hidden');
}
```

- [ ] **Step 7: Add button to header**

Add to header buttons:
```html
<button onclick="openMasteryModal()" class="text-sm bg-card/50 px-3 py-1 rounded-full hover:bg-yellow-600 transition-colors">🏆 Mastery</button>
```

- [ ] **Step 8: Update loadGame**

Add to loadGame:
```javascript
state.masteryLevels = state.masteryLevels || { egg: 0, coin: 0, pet: 0, event: 0 };
state.masteryXP = state.masteryXP || { egg: 0, coin: 0, pet: 0, event: 0 };
```

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add global mastery tracks system"
```

---

### Task 5: Integration & Testing

**Files:**
- Modify: `index.html` - verify all systems work together

- [ ] **Step 1: Verify XP flows correctly between systems**

Check that:
- Pet leveling gives XP from clicks, sells, and spawns
- Skill upgrades consume coins and update properly
- Mastery gains XP from relevant actions
- Evolution doesn't reset XP or levels

- [ ] **Step 2: Test persistence**

Save game, reload, verify:
- Pet levels persist
- Skill levels persist
- Mastery progress persists

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete Feature B - depth and progression systems"
```

---

## Summary

| Task | Changes | Lines Modified |
|------|---------|----------------|
| 1. Pet Leveling | State, XP functions, UI, modal | ~100 lines |
| 2. Skill Trees | Branch system, upgrade functions | ~80 lines |
| 3. Evolution Visuals | Enhanced borders, progress bars | ~20 lines |
| 4. Mastery Tracks | State, XP functions, modal, buttons | ~80 lines |
| 5. Integration | Testing and fixes | ~20 lines |
| **Total** | | **~300 lines** |

---

## Dependencies

- Task 1 is foundation for all other tasks
- Tasks 2, 3, 4 can be developed in parallel after Task 1
- Task 5 is final integration and testing

---

## Risks & Mitigations

1. **State bloat**: New properties may make localStorage saves large
   - Mitigate: All new state properties are optional with fallback defaults

2. **XP inflation**: Players may accumulate too much XP
   - Mitigate: XP requirements scale exponentially (1.5x per level)

3. **Complexity creep**: Too many systems competing for attention
   - Mitigate: Each system is self-contained and can be toggled off