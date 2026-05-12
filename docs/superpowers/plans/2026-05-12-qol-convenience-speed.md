# QoL: Convenience & Speed Features Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Quick Sell, Auto-Upgrade Toggle, Keyboard Shortcuts, and Batch Select for pets.

**Architecture:** Extend state with new properties for auto-upgrade settings and batch selection. Add mousedown/mouseup events for hold-to-sell. Add keydown listener for keyboard shortcuts. Implement multi-select UI in collection.

**Tech Stack:** Single HTML file, vanilla JavaScript, localStorage persistence

---

## File Structure

**Modify:**
- `index.html` - All changes in this single file

---

## Implementation Tasks

### Task 1: Auto-Upgrade Toggle

**Files:**
- Modify: `index.html:616-643` (state)
- Modify: `index.html:417-449` (auto-sell section - add near it)
- Modify: `index.html:1570-1601` (gameTick)

- [ ] **Step 1: Add autoUpgrade state**

Add to state object:
```javascript
autoUpgrade: {
    enabled: false,
    click: false,
    eggValue: false,
    autoClick: false,
    luck: false
}
```

- [ ] **Step 2: Add Auto-Upgrade section HTML**

Add after Auto-Sell section (~line 449):
```html
<!-- Auto-Upgrade Section -->
<div class="bg-card/60 backdrop-blur rounded-xl p-4 mb-6 border border-blue-500/20">
    <h3 class="font-bold mb-3 flex items-center gap-2">⬆️ Auto-Upgrade</h3>
    <div class="flex items-center justify-between mb-3">
        <div class="flex items-center gap-3">
            <label class="relative inline-flex items-center cursor-pointer">
                <input type="checkbox" id="autoUpgradeMasterToggle" class="sr-only peer" onchange="toggleAutoUpgradeMaster()">
                <div class="w-11 h-6 bg-gray-600 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-500"></div>
            </label>
            <span class="text-sm font-semibold" id="autoUpgradeMasterStatus">OFF</span>
        </div>
        <span class="text-xs text-gray-500">Auto buy upgrades when affordable</span>
    </div>
    <div class="grid grid-cols-4 gap-2 text-xs">
        <div class="flex items-center gap-1">
            <input type="checkbox" id="autoUpgradeClick" onchange="updateAutoUpgradeSettings()">
            <span>Click</span>
        </div>
        <div class="flex items-center gap-1">
            <input type="checkbox" id="autoUpgradeEggValue" onchange="updateAutoUpgradeSettings()">
            <span>Egg Val</span>
        </div>
        <div class="flex items-center gap-1">
            <input type="checkbox" id="autoUpgradeAutoClick" onchange="updateAutoUpgradeSettings()">
            <span>Auto+</span>
        </div>
        <div class="flex items-center gap-1">
            <input type="checkbox" id="autoUpgradeLuck" onchange="updateAutoUpgradeSettings()">
            <span>Luck</span>
        </div>
    </div>
</div>
```

- [ ] **Step 3: Add toggle and update functions**

Add after toggleAutoSell functions (~line 1318):
```javascript
function toggleAutoUpgradeMaster() {
    const toggle = document.getElementById('autoUpgradeMasterToggle');
    state.autoUpgrade.enabled = toggle.checked;
    updateAutoUpgradeUI();
    showToast(state.autoUpgrade.enabled ? '⬆️ Auto-Upgrade ON' : '⬆️ Auto-Upgrade OFF');
}

function updateAutoUpgradeSettings() {
    state.autoUpgrade.click = document.getElementById('autoUpgradeClick').checked;
    state.autoUpgrade.eggValue = document.getElementById('autoUpgradeEggValue').checked;
    state.autoUpgrade.autoClick = document.getElementById('autoUpgradeAutoClick').checked;
    state.autoUpgrade.luck = document.getElementById('autoUpgradeLuck').checked;
}

function updateAutoUpgradeUI() {
    document.getElementById('autoUpgradeMasterToggle').checked = state.autoUpgrade.enabled;
    document.getElementById('autoUpgradeMasterStatus').textContent = state.autoUpgrade.enabled ? 'ON' : 'OFF';
    document.getElementById('autoUpgradeMasterStatus').className = state.autoUpgrade.enabled ? 'text-sm font-semibold text-green-400' : 'text-sm font-semibold text-gray-400';
    document.getElementById('autoUpgradeClick').checked = state.autoUpgrade.click;
    document.getElementById('autoUpgradeEggValue').checked = state.autoUpgrade.eggValue;
    document.getElementById('autoUpgradeAutoClick').checked = state.autoUpgrade.autoClick;
    document.getElementById('autoUpgradeLuck').checked = state.autoUpgrade.luck;
}
```

- [ ] **Step 4: Add auto-upgrade check in gameTick**

In gameTick function (~line 1570), add before `checkEventExpiry()`:
```javascript
// Auto-upgrade checks
if (state.autoUpgrade.enabled) {
    if (state.autoUpgrade.click && state.coins >= getClickUpgradeCost()) {
        upgradeClick();
    }
    if (state.autoUpgrade.eggValue && state.coins >= getEggValueUpgradeCost()) {
        upgradeEggValue();
    }
    if (state.autoUpgrade.autoClick && state.coins >= getAutoClickUpgradeCost()) {
        upgradeAutoClick();
    }
    if (state.autoUpgrade.luck && state.coins >= getLuckUpgradeCost()) {
        upgradeLuck();
    }
}
```

- [ ] **Step 5: Update loadGame**

Add to loadGame:
```javascript
state.autoUpgrade = state.autoUpgrade || { enabled: false, click: false, eggValue: false, autoClick: false, luck: false };
updateAutoUpgradeUI();
```

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add auto-upgrade toggle system"
```

---

### Task 2: Quick Sell

**Files:**
- Modify: `index.html:290-292` (sell button)
- Add: mouse event handlers

- [ ] **Step 1: Update sell button with hold support**

Replace sell button (~line 290):
```html
<button id="quickSellBtn" onmousedown="startQuickSell()" onmouseup="stopQuickSell()" onmouseleave="stopQuickSell()" class="text-2xl mb-1">🏪</button>
```

- [ ] **Step 2: Add Quick Sell state**

Add to state:
```javascript
quickSell: { active: false, interval: null, count: 0 }
```

- [ ] **Step 3: Add Quick Sell functions**

Add after closeMasteryModal (~line 1806):
```javascript
function startQuickSell() {
    if (state.quickSell.active) return;
    state.quickSell.active = true;
    state.quickSell.count = 0;
    state.quickSell.interval = setInterval(() => {
        if (state.eggs >= 100) {
            // Temporarily bypass toast for rapid sell
            const originalEggs = state.eggs;
            const originalCoins = state.coins;
            sellEggsSilent();
            state.quickSell.count++;
            // Show progress every 10 sells
            if (state.quickSell.count % 10 === 0) {
                const coinsEarned = state.coins - originalCoins;
                showToastQuickSell(state.quickSell.count, coinsEarned);
            }
        } else {
            stopQuickSell();
            if (state.quickSell.count > 0) {
                showToast('🏪 Quick Sell: ' + state.quickSell.count + ' batches sold!');
            }
        }
    }, 200); // Sell every 200ms while holding
}

function stopQuickSell() {
    if (state.quickSell.interval) {
        clearInterval(state.quickSell.interval);
        state.quickSell.interval = null;
    }
    state.quickSell.active = false;
}

function sellEggsSilent() {
    // Silent version of sellEggs without toast
    const level = state.clickLevel;
    if (state.eggs >= 10000 && level >= 25) {
        state.eggs -= 10000;
        const coinsEarned = 2000 * (1 + state.eggValueLevel * 0.1) * (state.event.active ? 2 : 1);
        state.coins += coinsEarned;
        state.stats.totalCoinsEarned += coinsEarned;
        state.pets.forEach(pet => addPetXP(pet, Math.floor(coinsEarned * 0.1)));
        addMasteryXP('coin', Math.floor(coinsEarned * 0.2));
    } else if (state.eggs >= 1000 && level >= 10) {
        state.eggs -= 1000;
        const coinsEarned = 150 * (1 + state.eggValueLevel * 0.1) * (state.event.active ? 2 : 1);
        state.coins += coinsEarned;
        state.stats.totalCoinsEarned += coinsEarned;
        state.pets.forEach(pet => addPetXP(pet, Math.floor(coinsEarned * 0.1)));
        addMasteryXP('coin', Math.floor(coinsEarned * 0.2));
    } else if (state.eggs >= 100) {
        state.eggs -= 100;
        const coinsEarned = 10 * (1 + state.eggValueLevel * 0.1) * (state.event.active ? 2 : 1);
        state.coins += coinsEarned;
        state.stats.totalCoinsEarned += coinsEarned;
        state.pets.forEach(pet => addPetXP(pet, Math.floor(coinsEarned * 0.1)));
        addMasteryXP('coin', Math.floor(coinsEarned * 0.2));
    }
    checkQuestProgress('sellPremium');
    checkQuestProgress('earnCoins', coinsEarned || 0);
}

function showToastQuickSell(count, coins) {
    // Minimal toast during quick sell
    const toast = document.getElementById('toast');
    document.getElementById('toastText').textContent = 'Sold x' + count + ' (+' + Math.floor(coins) + ' 🪙)';
    toast.classList.remove('hidden');
    setTimeout(() => toast.classList.add('hidden'), 500);
}
```

- [ ] **Step 4: Update loadGame**

Add to loadGame:
```javascript
state.quickSell = state.quickSell || { active: false, interval: null, count: 0 };
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add quick sell hold-to-sell feature"
```

---

### Task 3: Keyboard Shortcuts

**Files:**
- Modify: `index.html:1732-1737` (init section)

- [ ] **Step 1: Add keyboard listener**

In init section (~line 1732), add:
```javascript
document.addEventListener('keydown', handleKeyboardShortcuts);

function handleKeyboardShortcuts(e) {
    // Ignore if typing in input
    if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;

    switch(e.key) {
        case '1':
            clickEgg();
            break;
        case '2':
            openSpinModal();
            break;
        case '3':
            checkDailyReward();
            break;
        case '4':
            sellEggs();
            break;
        case 'Escape':
            // Close any open modal
            document.querySelectorAll('.fixed.inset-0').forEach(modal => {
                if (!modal.classList.contains('hidden')) {
                    modal.classList.add('hidden');
                }
            });
            break;
    }
}
```

- [ ] **Step 2: Add hint in UI**

Add small hint text near header:
```html
<p class="text-xs text-gray-500 mt-1">Keys: 1=Click, 2=Spin, 3=Daily, 4=Sell, Esc=Close</p>
```

Or add as tooltip/floating hint that can be toggled.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add keyboard shortcuts"
```

---

### Task 4: Batch Select for Pet Actions

**Files:**
- Modify: `index.html:616-643` (state)
- Modify: `index.html:1645-1659` (renderCollection)
- Modify: `index.html:505-521` (trade section)
- Modify: `index.html:514-521` (fusion section)

- [ ] **Step 1: Add selectedPets state**

Add to state:
```javascript
selectedPets: [],    // Array of pet emojis currently selected
batchMode: false    // Whether batch selection mode is active
```

- [ ] **Step 2: Update renderCollection for batch selection**

Modify renderCollection to add selection UI:
```javascript
// Add selection indicator and click handler
const isSelected = state.selectedPets.includes(pet);
const selectionClass = isSelected ? 'ring-2 ring-yellow-400 ring-offset-2' : '';

<div onclick="handlePetClick('${pet}', event)" class="pet-card slide-in cursor-pointer bg-gradient-to-br from-indigo-900/50 to-purple-900/30 rounded-xl p-2 text-center border ${borderClass} ${selectionClass} ${i % 3 === 0 ? 'pet-bounce' : ''}" ...>

${isSelected ? '<span class="absolute top-0 right-0 bg-yellow-400 text-black text-xs rounded-full w-5 h-5 flex items-center justify-center">✓</span>' : ''}
```

- [ ] **Step 3: Add handlePetClick function**

Add after openLevelModal (~line 1003):
```javascript
function handlePetClick(pet, event) {
    if (event.shiftKey && state.selectedPets.length > 0) {
        // Range select - select all pets between last selected and this one
        const lastSelected = state.selectedPets[state.selectedPets.length - 1];
        const lastIndex = state.pets.indexOf(lastSelected);
        const currentIndex = state.pets.indexOf(pet);
        const start = Math.min(lastIndex, currentIndex);
        const end = Math.max(lastIndex, currentIndex);
        for (let i = start; i <= end; i++) {
            if (!state.selectedPets.includes(state.pets[i])) {
                state.selectedPets.push(state.pets[i]);
            }
        }
    } else if (event.ctrlKey || event.metaKey) {
        // Toggle selection
        if (state.selectedPets.includes(pet)) {
            state.selectedPets = state.selectedPets.filter(p => p !== pet);
        } else {
            state.selectedPets.push(pet);
        }
    } else {
        // Normal click - open training modal (existing behavior)
        openTrainingModal(pet);
    }
    renderCollection();
}

function clearPetSelection() {
    state.selectedPets = [];
    renderCollection();
}
```

- [ ] **Step 4: Add batch action buttons in trade/fusion sections**

In tradeSection (~line 505):
```html
<div id="tradeSection" class="text-center text-gray-500 py-4">
    ${state.selectedPets.length > 0 ? `
        <div class="mb-3 flex justify-center gap-2">
            <button onclick="tradeSelectedPets()" class="bg-cyan-600 hover:bg-cyan-500 px-4 py-2 rounded text-sm">
                Trade ${state.selectedPets.length} Pets
            </button>
            <button onclick="clearPetSelection()" class="bg-gray-600 hover:bg-gray-500 px-4 py-2 rounded text-sm">
                Clear Selection
            </button>
        </div>
    ` : ''}
    [existing trade content]
</div>
```

- [ ] **Step 5: Add tradeSelectedPets function**

Add after tradePet function:
```javascript
function tradeSelectedPets() {
    let totalCoins = 0;
    state.selectedPets.forEach(pet => {
        if (state.petCounts[pet] > 1) {
            const tradeValue = 25;
            state.petCounts[pet]--;
            totalCoins += tradeValue;
            if (state.petCounts[pet] === 0) {
                state.pets = state.pets.filter(p => p !== pet);
            }
        }
    });
    state.coins += totalCoins;
    state.stats.totalCoinsEarned += totalCoins;
    showToast('🔄 Traded ' + state.selectedPets.length + ' pets for ' + totalCoins + ' 🪙');
    state.selectedPets = [];
    updateTradeSection();
    renderCollection();
    updateUI();
}
```

- [ ] **Step 6: Add fuseSelectedPets function**

Add after fusePet:
```javascript
function fuseSelectedPets() {
    let fused = 0;
    state.selectedPets.forEach(pet => {
        if (state.petCounts[pet] >= 2) {
            const evoLevel = state.petEvolution[pet] || 0;
            if (evoLevel < 3) {
                state.petCounts[pet] -= 2;
                state.petEvolution[pet] = evoLevel + 1;
                if (state.petCounts[pet] === 0) {
                    state.pets = state.pets.filter(p => p !== pet);
                    delete state.petCounts[pet];
                }
                fused++;
            }
        }
    });
    if (fused > 0) {
        showToast('🔥 Fused ' + fused + ' pets!');
        updateFusionSection();
        renderCollection();
        updateUI();
    }
    state.selectedPets = [];
}
```

- [ ] **Step 7: Update loadGame**

Add to loadGame:
```javascript
state.selectedPets = state.selectedPets || [];
state.batchMode = state.batchMode || false;
```

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add batch select for pet actions"
```

---

### Task 5: Integration Testing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Verify all features work together**
- [ ] **Step 2: Verify persistence works**
- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete QoL convenience features"
```

---

## Summary

| Task | Changes |
|------|---------|
| 1. Auto-Upgrade Toggle | Toggle, per-upgrade checks, game loop integration |
| 2. Quick Sell | Hold-to-sell with interval, silent sell function |
| 3. Keyboard Shortcuts | Keydown listener, 1-4 and Escape bindings |
| 4. Batch Select | Multi-select state, range/ctrl click, bulk actions |
| 5. Integration | Testing and final commit |

---

## Risks & Mitigations

1. **Quick Sell spam**: Sell interval could be too fast
   - Mitigate: 200ms interval is reasonable, shows progress

2. **Auto-upgrade feedback loop**: Could cause rapid upgrades
   - Mitigate: Only triggers when coins >= cost, 1 upgrade per tick

3. **Keyboard conflicts**: Could conflict with browser shortcuts
   - Mitigate: Only uses 1-4, Escape; ignores when in input fields