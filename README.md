// Monster Mod for Sandboxels
// Version 1.0
// Adds a few simple monsters

// Make sure elements is defined
if (typeof elements === "undefined") {
    throw new Error("Sandboxels: elements object not found. Load this as a mod inside Sandboxels.");
}

// Utility: check if an element name exists
function hasElement(name) {
    return Object.prototype.hasOwnProperty.call(elements, name);
}

// -----------------
// SLIME MONSTER
// -----------------

elements.monster_slime = {
    color: [
        "#4ac94a",
        "#3fb63f",
        "#35a235"
    ],
    behavior: [
        "XX|CR:monster_slime%0.2|XX",
        "M2|XX|M2",
        "XX|M1|XX"
    ],
    // Spreads and converts sand and dirt into more slime
    tick: function(pixel) {
        // Basic life decay (optional)
        if (!pixel.age) pixel.age = 0;
        pixel.age++;

        // Convert nearby sand / dirt into more slime
        const convertTargets = ["sand", "dirt", "wet_sand", "mud"];

        for (let dx = -1; dx <= 1; dx++) {
            for (let dy = -1; dy <= 1; dy++) {
                if (dx === 0 && dy === 0) continue;
                const x = pixel.x + dx;
                const y = pixel.y + dy;
                if (!isEmpty(x, y, true)) {
                    const p = pixelMap[x][y];
                    if (p && convertTargets.includes(p.element) && Math.random() < 0.05) {
                        changePixel(p, "monster_slime");
                    }
                }
            }
        }

        // Slime slowly dries out near heat
        if (pixel.temp > 60 && Math.random() < 0.02) {
            changePixel(pixel, "dust");
        }
    },
    category: "life",
    state: "solid",
    density: 1000,
    tempHigh: 120,
    stateHigh: "steam",
    breakInto: "slime",
    reactions: {
        "salt": { elem2: "salt_water", chance: 0.3 },
    },
    desc: "A gooey monster that spreads over earthy materials and slowly dries out from heat."
};

// -----------------
// FIRE ELEMENTAL
// -----------------

elements.monster_fire_elemental = {
    color: ["#ff7a00", "#ffbf00", "#ff4500"],
    behavior: [
        "M1|M1|M1",
        "M1|XX|M1",
        "M1|M1|M1"
    ],
    // Ignites nearby flammable elements and is hurt by water
    tick: function(pixel) {
        // Keep hot
        if (pixel.temp < 800) pixel.temp = 800;

        // Ignite flammable neighbors
        const flammable = [
            "wood", "plant", "grass", "paper", "coal", "oil",
            "gunpowder", "fireworks", "rubber", "plastic"
        ];

        for (let dx = -1; dx <= 1; dx++) {
            for (let dy = -1; dy <= 1; dy++) {
                if (dx === 0 && dy === 0) continue;
                const x = pixel.x + dx;
                const y = pixel.y + dy;
                if (!isEmpty(x, y, true)) {
                    const p = pixelMap[x][y];
                    if (p && flammable.includes(p.element) && Math.random() < 0.3) {
                        // Try to change to fire if it exists
                        if (hasElement("fire")) {
                            changePixel(p, "fire");
                        }
                    }
                }
            }
        }

        // Water hurts / kills it
        const waterTypes = ["water", "salt_water", "dirty_water", "ice", "snow"];
        for (let dx = -1; dx <= 1; dx++) {
            for (let dy = -1; dy <= 1; dy++) {
                const x = pixel.x + dx;
                const y = pixel.y + dy;
                if (!isEmpty(x, y, true)) {
                    const p = pixelMap[x][y];
                    if (p && waterTypes.includes(p.element)) {
                        // Both disappear into steam
                        if (hasElement("steam")) {
                            changePixel(pixel, "steam");
                            changePixel(p, "steam");
                        } else {
                            deletePixel(pixel.x, pixel.y);
                            deletePixel(p.x, p.y);
                        }
                        return;
                    }
                }
            }
        }
    },
    category: "life",
    state: "gas",
    density: 1,
    temp: 900,
    tempHigh: 2000,
    stateHigh: "plasma",
    conduct: 0.3,
    desc: "A living flame that wanders and ignites flammable materials, but is destroyed by water."
};

// -----------------
// STONE GOLEM
// -----------------

elements.monster_stone_golem = {
    color: ["#555555", "#666666", "#777777"],
    behavior: [
        "XX|M1|XX",
        "M1|XX|M1",
        "XX|M1|XX"
    ],
    // Slow, heavy, and resistant; can be cracked by explosions
    tick: function(pixel) {
        // Very slow movement: occasionally skips move
        if (Math.random() < 0.7) {
            return; // 70% of the time, do nothing (slow)
        }

        // Self-repair if near stone
        const repairSources = ["rock", "stone", "boulder"];
        for (let dx = -1; dx <= 1; dx++) {
            for (let dy = -1; dy <= 1; dy++) {
                const x = pixel.x + dx;
                const y = pixel.y + dy;
                if (!isEmpty(x, y, true)) {
                    const p = pixelMap[x][y];
                    if (p && repairSources.includes(p.element) && Math.random() < 0.05) {
                        if (pixel.health === undefined) pixel.health = 100;
                        pixel.health = Math.min(100, pixel.health + 5);
                    }
                }
            }
        }

        // Explosion damage handled via reactions below
    },
    category: "life",
    state: "solid",
    density: 2600,
    hardness: 0.9,
    tempHigh: 1600,
    stateHigh: "magma",
    breakInto: "rock",
    reactions: {
        // Explosions crack golem into rock / gravel
        "explosion": {
            elem1: "rock",
            chance: 0.7
        },
        "acid": {
            elem1: "mud",
            chance: 0.2
        }
    },
    desc: "A slow-moving living statue of stone. Very tough, but vulnerable to powerful explosions and extreme heat."
};

// -----------------
// OPTIONAL: CUSTOM CATEGORY
// -----------------

// If you want a separate monster tab/category, you can redefine category like this:
// elements.monster_slime.category = "monsters";
// elements.monster_fire_elemental.category = "monsters";
// elements.monster_stone_golem.category = "monsters";
//
// Then in Sandboxels’ main code you’d add a `monsters` category if it doesn’t
// already exist. If you’re only using modding from the UI, leaving them under
// `life` is usually simpler.

console.log("Monster Mod loaded: monster_slime, monster_fire_elemental, monster_stone_golem");
