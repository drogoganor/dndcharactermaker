<template>
  <div class='bd-main-container container'>
    <div class='content'>
      <div class='title'>D&amp;D 5e Character Sheet Maker</div>
      <div class='box'>
        This tool will generate a pre-filled
        <a href="https://dnd.wizards.com/articles/features/character_sheets">D&amp;D 5e character sheet</a> in PDF format. You'll need a
        <a href="https://dnd.wizards.com/products/tabletop-games/rpg-products/rpg_playershandbook">Player's Handbook</a> to fill out some details from page number references.
      </div>
      <div class="box">
        This is a WIP. Numerous features are incomplete or missing. Please see the
        <a href="https://github.com/drogoganor/dndcharactermaker" class="alert-link">GitHub page</a> for more details or to report issues.
      </div>
    </div>
    <div class='content has-text-left'>
      <div class='columns'>
        <div class='column is-2'>Level:</div>
        <div class='column is-1' id="level">{{ level }}</div>
      </div>
      <div class='columns'>
        <div class='column is-2'>XP:</div>
        <input class='input column is-2' id="xp" type="text" v-model="output.xp">
      </div>
      <div class='columns'>
        <div class='column is-2'>Character Name:</div>
        <input class='input column is-4' id="charname" type="text" v-model="output.characterName">
      </div>
      <div class='columns'>
        <div class='column is-2'>Player Name:</div>
        <input class='input column is-4' id="playername" type="text" v-model="output.playerName">
      </div>
      <div>
        <div>Race:</div>
        <div id="character-race">
          <div class="buttons are-small has-addons">
            <button
              type="button"
              v-for="race in races"
              v-bind:key="race.id"
              v-on:click="setRace(race)"
              :class="['button', { 'is-success is-selected': output.race === race.text }]"
            >{{ race.text }}</button>
          </div>
        </div>
      </div>
      <div>
        <div>Class:</div>
        <div id="character-class">
          <div class="buttons are-small has-addons">
            <button
              type="button"
              v-for="cls in classes"
              v-bind:key="cls.id"
              v-on:click="setClass(cls)"
              :class="['button', { 'is-success is-selected': output.class === cls.text }]"
            >{{ cls.text }}</button>
          </div>
        </div>
      </div>
      <div>
        <div>Law/Chaos:</div>
        <div id="character-alignment1">
          <div class="buttons are-small has-addons">
            <button
              type="button"
              v-for="align in alignmentLawChaos"
              v-bind:key="align.id"
              v-on:click="output.alignmentLawChaos = align.text"
              :class="['button', { 'is-success is-selected': output.alignmentLawChaos === align.text }]"
            >{{ align.text }}</button>
          </div>
        </div>
      </div>
      <div>
        <div>Good/Evil:</div>
        <div id="character-alignment2">
          <div class="buttons are-small has-addons">
            <button
              type="button"
              v-for="align in alignmentGoodEvil"
              v-bind:key="align.id"
              v-on:click="output.alignmentGoodEvil = align.text"
              :class="['button', { 'is-success is-selected': output.alignmentGoodEvil === align.text }]"
            >{{ align.text }}</button>
          </div>
        </div>
      </div>
      <div>
        <div>Background:</div>
        <div id="character-background">
          <div class="buttons are-small has-addons">
            <button
              type="button"
              v-for="bg in backgrounds"
              v-bind:key="bg.id"
              v-on:click="setBackground(bg)"
              :class="['button', { 'is-success is-selected': output.background === bg.text }]"
            >{{ bg.text }}</button>
          </div>
        </div>
      </div>
      <div class='columns' v-if="toolProficienciesText !== ''">
        <div class='column is-2'>Tool Proficiencies:</div>
        <div class='column' id="toolProfs">{{ toolProficienciesText }}</div>
      </div>
      <div class='columns'>
        <div class='column is-2'>Currency:</div>
        <div class='column' id="currency">{{ currencyText }}</div>
      </div>
      <div>
        <div>Stats:</div>
      </div>
      <div class='columns'>
        <div class='column is-2'>Reset Stats:</div>
        <div class='column'>
          <button type="button" class='button is-danger' v-on:click="standardStats">Standard Array</button>&nbsp;
          <button type="button" class='button is-danger' v-on:click="rollStats">Reroll</button>
        </div>
      </div>
      <div class='columns' v-if='statRolls.length > 0'>
        <div class='column is-2'>Assign <b>{{ currentStatAssignmentText }}</b>:</div>
        <div class='column buttons'>
          <button
            type="button"
            class="button is-success"
            v-for="(r, i) in statRolls"
            v-bind:key="i"
            v-on:click="allocateStat(r)"
          >{{ r }}</button>
        </div>
      </div>
      <div class='columns'>
        <div class='column is-1'></div>
        <div class='column is-1'>Assigned</div>
        <div class='column is-1'>Racial</div>
        <div class='column is-1'>Total</div>
        <div class='column is-1'>Modifier</div>
      </div>
      <div class='columns' id="stat-block" v-for="block in statBlocks" v-bind:key="block.id">
        <div class='column is-1'>{{ block.text }}</div>
        <div class='column is-1'>{{ output.statArray[block.id] }}</div>
        <div class='column is-1'>{{ output.raceStatBonuses[block.id] }}</div>
        <div class='column is-1'>{{ output.finalStats[block.id] }}</div>
        <div class='column is-1'>{{ formatModifier(output.statModifiers[block.id]) }}</div>
      </div>
      <div class='columns'>
        <div class='column is-2'>Proficiencies:</div>
        <div class='column' id="proficiencies">
          <div class='tags are-small'>
            <span
              v-for="prof in output.proficiencies"
              v-bind:key="prof"
              class='tag'
              role="alert"
            >{{ skills[prof].text }}</span>
          </div>
          <div v-if="allProficienciesChosen">
            <button type="button" class='button is-danger' v-on:click="resetProficiencies">Reset Proficiencies</button>
          </div>
          <div v-if="!allProficienciesChosen">
            <div>{{ proficiencesLeftText }}</div>
            <div class='buttons are-small'>
              <button
                type="button"
                class='button is-success'
                v-for="prof in availableProficiencies"
                v-bind:key="prof.id"
                v-on:click="addProficiency(prof.id)"
              >{{ prof.text }}</button>
            </div>
          </div>
        </div>
      </div>

      <div class='columns'>
        <div class='column is-2'>Equipment:</div>
        <div class='column tags are-small'>
          <div class='tags are-small'>
          <span v-for="eq in equipmentTextList" v-bind:key="eq" class='tag'>{{ eq }}</span>
          </div>
          <div v-if="hasChosenAnyEquipment">
            <button type="button" class='button is-danger' v-on:click="resetEquipment">Reset Equipment</button>
          </div>
          <div v-for="choices in equipmentChoiceModel" v-bind:key="choices.id">
              <span v-for="choice in choices.choices" v-bind:key="choice.id" class='buttons are-small'>
                  <span class='tag'>{{ choice.num }}x</span>
                  <button type="button" v-for="item in choice.items" v-bind:key="item.id" class='button is-success' v-on:click="selectEquipment(choices.id, choice.id, item)">{{ getEquipmentName(item) }}</button>
                  <span class='tag' v-if="choice.id < choices.choices.length - 1">OR</span>
              </span>
              <hr />
          </div>
        </div>
      </div>

      <div class='columns'>
        <div class='column is-2'>Languages:</div>
        <div class='column tags are-small'>
          <div class='tags are-small'>
            <span
              v-for="lang in output.languageids"
              v-bind:key="lang"
              class='tag'
            >{{ languages[lang].text }}</span>
          </div>
          <div v-if="hasBonusLanguages && allLanguagesChosen">
            <button type="button" class='button is-danger' v-on:click="resetLanguages">Reset Languages</button>
          </div>
          <div v-if="!allLanguagesChosen">
            <div>{{ languagesLeftText }}</div>
            <div class='buttons are-small'>
              <button
                type="button"
                class='button is-success'
                v-for="lang in availableLanguages"
                v-bind:key="lang.id"
                v-on:click="addLanguage(lang.id)"
              >{{ lang.text }}</button>
            </div>
          </div>
        </div>
      </div>

      <div id="finish">
        <button
          type="button"
          id="generate"
          class='button is-success is-large'
          v-on:click="generate"
          v-bind="generateButtonBindings"
        >Generate PDF</button>
      </div>

      <div>&nbsp;</div>
    </div>
  </div>
</template>


<script>
import { saveAs } from 'file-saver'

export default {
  name: 'DnD',
  data: function () {
    return {
      statAssignmentIndex: 0,
      xpLevels: [
        { xp: 0, level: 1 },
        { xp: 300, level: 2 },
        { xp: 900, level: 3 },
        { xp: 2700, level: 4 },
        { xp: 6500, level: 5 },
        { xp: 14000, level: 6 },
        { xp: 23000, level: 7 },
        { xp: 34000, level: 8 },
        { xp: 48000, level: 9 },
        { xp: 64000, level: 10 },
        { xp: 85000, level: 11 },
        { xp: 100000, level: 12 },
        { xp: 120000, level: 13 },
        { xp: 140000, level: 14 },
        { xp: 165000, level: 15 },
        { xp: 195000, level: 16 },
        { xp: 225000, level: 17 },
        { xp: 265000, level: 18 },
        { xp: 305000, level: 19 },
        { xp: 355000, level: 20 },
      ],
      races: [
        { id: 0, raceid: 0, text: 'Dwarf' },
        { id: 1, raceid: 0, text: 'Hill Dwarf' },
        { id: 2, raceid: 0, text: 'Mountain Dwarf' },
        { id: 3, raceid: 1, text: 'Elf' },
        { id: 4, raceid: 1, text: 'High Elf' },
        { id: 5, raceid: 1, text: 'Wood Elf' },
        { id: 6, raceid: 1, text: 'Dark Elf (Drow)' },
        { id: 7, raceid: 2, text: 'Halfling' },
        { id: 8, raceid: 2, text: 'Halfling (Lightfoot)' },
        { id: 9, raceid: 2, text: 'Halfling (Stout)' },
        { id: 10, raceid: 3, text: 'Human' },
        { id: 11, raceid: 4, text: 'Dragonborn' },
        { id: 12, raceid: 5, text: 'Gnome' },
        { id: 13, raceid: 5, text: 'Forest Gnome' },
        { id: 14, raceid: 5, text: 'Rock Gnome' },
        { id: 15, raceid: 6, text: 'Half-Elf' },
        { id: 16, raceid: 7, text: 'Half-Orc' },
        { id: 17, raceid: 8, text: 'Tiefling' },
      ],
      classes: [
        { id: 0, text: 'Barbarian' },
        { id: 1, text: 'Bard' },
        { id: 2, text: 'Cleric' },
        { id: 3, text: 'Druid' },
        { id: 4, text: 'Fighter' },
        { id: 5, text: 'Monk' },
        { id: 6, text: 'Paladin' },
        { id: 7, text: 'Ranger' },
        { id: 8, text: 'Rogue' },
        { id: 9, text: 'Sorcerer' },
        { id: 10, text: 'Warlock' },
        { id: 11, text: 'Wizard' },
      ],
      armorCategories: [
        { id: 0, text: 'Light Armor' },
        { id: 1, text: 'Medium Armor' },
        { id: 2, text: 'Heavy Armor' },
        { id: 3, text: 'Shield' },
      ],
      classArmorProficiencies: [
        { id: 0, profs: [0, 1, 3] },
        { id: 1, profs: [0] },
        { id: 2, profs: [0, 1, 3] },
        { id: 3, profs: [0, 1, 3] }, // Non-metal only
        { id: 4, profs: [0, 1, 2, 3] },
        { id: 5, profs: [] },
        { id: 6, profs: [0, 1, 2, 3] },
        { id: 7, profs: [0, 1, 3] },
        { id: 8, profs: [0] },
        { id: 9, profs: [] },
        { id: 10, profs: [0] },
        { id: 11, profs: [] },
      ],
      classWeaponProficiencies: [
        { id: 0, categories: [0, 1], weapons: [] }, // categories: Weapon categories and weapons: manually listed weapons
        { id: 1, categories: [0], weapons: [33, 21, 25, 27] },
        { id: 2, categories: [0], weapons: [] },
        { id: 3, categories: [], weapons: [0, 1, 11, 4, 6, 7, 26, 8, 13, 9] },
        { id: 4, categories: [0, 1], weapons: [] },
        { id: 5, categories: [0], weapons: [27] },
        { id: 6, categories: [0, 1], weapons: [] },
        { id: 7, categories: [0, 1], weapons: [] },
        { id: 8, categories: [0], weapons: [33, 21, 25, 27] },
        { id: 9, categories: [], weapons: [1, 11, 13, 7, 10] },
        { id: 10, categories: [0], weapons: [] },
        { id: 11, categories: [0], weapons: [1, 11, 13, 7, 10] },
      ],
      equipmentCategories: [
        { id: 0, text: 'Weapon' },
        { id: 1, text: 'Armor' },
        { id: 2, text: 'Pack' },
        { id: 3, text: 'Shield' },
        { id: 4, text: 'Ammo' },
        { id: 5, text: 'Trinket' },
        { id: 6, text: 'Other' },
      ],
      weaponCategories: [
        { id: 0, text: 'Simple weapons' },
        { id: 1, text: 'Martial weapons' },
      ],
      damageTypes: [
        { id: 0, text: 'Bludgeoning' },
        { id: 1, text: 'Piercing' },
        { id: 2, text: 'Slashing' },
      ],
      classEquipment: [
        {
          id: 0, // Barbarian
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [17] }, // Greataxe, or
                { id: 1, num: 1, items: [14, 15, 16, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31] }, // Any martial melee
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 2, items: [3] }, // Handaxe, or
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13] }, // Any simple weapon
              ],
            },
          ],
          fixedEquip: [
            { num: 4, id: 4 }, // Javelin
            { num: 1, id: 37 }, // Explorer's pack
          ],
        },
        {
          id: 1, // Bard
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [25] }, // Rapier, or
                { id: 1, num: 1, items: [21] }, // Longsword, or
                { id: 2, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [38] }, // Diplomat's pack, or
                { id: 1, num: 1, items: [39] }, // Entertainer's pack
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [62] }, // Lute, or
                { id: 1, num: 1, items: [63] }, // Other musical instrument
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 1 }, // Dagger
            { num: 1, id: 45 }, // Leather armor
          ],
        },
        {
          id: 2, // Cleric
          equipChoices: [
            {
              id: 0,
              // * If proficient
              choices: [
                { id: 0, num: 1, items: [6] }, // Mace, or
                { id: 1, num: 1, items: [30] }, // Warhammer
              ],
            },
            {
              id: 1,
              // * If proficient
              choices: [
                { id: 0, num: 1, items: [49] }, // Scale mail, or
                { id: 1, num: 1, items: [45] }, // Leather armor, or
                { id: 2, num: 1, items: [53] }, // Chain mail
              ],
            },
            {
              id: 2,
              choices: [
                {
                  id: 0, num: 1, items: [10], extras: [
                    { id: 59, num: 20 }
                  ],
                }, // Light crossbow and 20 bolts, or (TODO: Special ammo modeling)
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 3,
              choices: [
                { id: 0, num: 1, items: [40] }, // Priest's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 57 }, // Shield
            { num: 1, id: 66 }, // Holy symbol
          ],
        },
        {
          id: 3, // Druid
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [56] }, // Wooden shield, or
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [26] }, // Scimitar, or
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] }, // Any simple melee weapon
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 45 }, // Leather armor
            { num: 1, id: 37 }, // Explorer's pack
            { num: 1, id: 65 }, // Druidic focus
          ],
        },
        {
          id: 4, // Fighter
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [53] }, // Chain mail, or
                {
                  id: 1, num: 1, items: [45], extras: [
                    { id: 35 },
                    { id: 58, num: 20 },
                  ]
                }, // Leather armor, longbow, and 20 arrows (TODO: Special ammo modeling)
              ],
            },
            {
              id: 1,
              choices: [
                {
                  id: 0, num: 1, items: [14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36],
                  extras: [{ id: 57 }]
                }, // A martial weapon and a shield, or
                { id: 1, num: 2, items: [14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36] }, // Two martial weapons
              ],
            },
            {
              id: 2,
              choices: [
                {
                  id: 0, num: 1, items: [10], extras: [
                    { id: 59, num: 20 }
                  ],
                }, // Light crossbow and 20 bolts, or (TODO: Special ammo modeling)
                { id: 1, num: 2, items: [3] }, // Two handaxes
              ],
            },
            {
              id: 3,
              choices: [
                { id: 0, num: 1, items: [41] }, // Dungeoneer's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
        },
        {
          id: 5, // Monk
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [27] }, // Shortsword, or
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [41] }, // Dungeoneer's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 20, id: 11 }, // 20 darts
          ],
        },
        {
          id: 6, // Paladin
          equipChoices: [
            {
              id: 0,
              choices: [
                {
                  id: 0, num: 1, items: [14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36],
                  extras: [{ id: 57 }]
                }, // A martial weapon and a shield, or
                { id: 1, num: 2, items: [14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36] }, // Two martial weapons
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 5, items: [4] }, // 5 Javelins, or
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] }, // Any simple melee weapon
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [40] }, // Priest's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 53 }, // Chain mail
            { num: 1, id: 66 }, // Holy symbol
          ],
        },
        {
          id: 7, // Ranger
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [49] }, // Scale mail, or
                { id: 1, num: 1, items: [45] }, // Leather armor
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 2, items: [27] }, // 2 Shortswords, or
                { id: 1, num: 2, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] }, // 2 simple melee weapons
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [41] }, // Dungeoneer's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 35 }, // Longbow
            { num: 20, id: 58 }, // 20 arrows
          ],
        },
        {
          id: 8, // Rogue
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [25] }, // Rapier, or
                { id: 1, num: 1, items: [27] }, // Shortsword
              ],
            },
            {
              id: 1,
              choices: [
                {
                  id: 0, num: 1, items: [12], extras: [
                    { id: 58, num: 20 }
                  ],
                }, // Shortbow and 20 arrows, or (TODO: Special ammo modeling)
                { id: 1, num: 1, items: [27] }, // Shortsword
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [42] }, // Burglar's pack, or
                { id: 1, num: 1, items: [41] }, // Dungeoneer's pack, or
                { id: 2, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 45 }, // Leather armor
            { num: 2, id: 1 }, // 2 daggers
            { num: 1, id: 106 }, // Thieves' tools
          ],
        },
        {
          id: 9, // Sorcerer
          equipChoices: [
            {
              id: 0,
              choices: [
                {
                  id: 0, num: 1, items: [10], extras: [
                    { id: 59, num: 20 }
                  ],
                }, // Light crossbow and 20 bolts, or (TODO: Special ammo modeling)
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [64] }, // Component pouch, or
                { id: 1, num: 1, items: [107] }, // Arcane focus
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [41] }, // Dungeoneer's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 2, id: 1 }, // 2 daggers
          ],
        },
        {
          id: 10, // Warlock
          equipChoices: [
            {
              id: 0,
              choices: [
                {
                  id: 0, num: 1, items: [10], extras: [
                    { id: 59, num: 20 }
                  ],
                }, // Light crossbow and 20 bolts, or (TODO: Special ammo modeling)
                { id: 1, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [64] }, // Component pouch, or
                { id: 1, num: 1, items: [107] }, // Arcane focus
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [43] }, // Scholar's pack, or
                { id: 1, num: 1, items: [41] }, // Dungeoneer's pack
              ],
            },
            {
              id: 3,
              choices: [ // TODO: This should really be fixed equipment, remember to model a single choice
                { id: 0, num: 1, items: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13] }, // Any simple weapon
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 45 }, // Leather armor
            { num: 2, id: 1 }, // 2 daggers
          ],
        },
        {
          id: 11, // Wizard
          equipChoices: [
            {
              id: 0,
              choices: [
                { id: 0, num: 1, items: [7] }, // Quarterstaff, or
                { id: 1, num: 1, items: [1] }, // Dagger
              ],
            },
            {
              id: 1,
              choices: [
                { id: 0, num: 1, items: [64] }, // Component pouch, or
                { id: 1, num: 1, items: [107] }, // Arcane focus
              ],
            },
            {
              id: 2,
              choices: [
                { id: 0, num: 1, items: [43] }, // Scholar's pack, or
                { id: 1, num: 1, items: [37] }, // Explorer's pack
              ],
            },
          ],
          fixedEquip: [
            { num: 1, id: 108 }, // Spellbook
          ],
        },
      ],
      equipment: [ // TODO: Finesse, cost, versatile, loading, heavy, reach, special
        {
          id: 0,
          text: 'Club',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d4',
          weapon: true,
          melee: true, // Whether it can be used melee (STR mod)
          thrown: false, // Whether it can be thrown if necessary. (DEX mod)
          light: true,
          twohanded: false,
          weight: 2
        },
        {
          id: 1,
          text: 'Dagger',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d4',
          weapon: true,
          melee: true,
          thrown: true,
          light: true,
          twohanded: false,
          weight: 1
        },
        {
          id: 2,
          text: 'Greatclub',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 10
        },
        {
          id: 3,
          text: 'Handaxe',
          type: 0,
          weaponCategory: 0,
          damage: 2,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: true,
          light: true,
          twohanded: false,
          weight: 2
        },
        {
          id: 4,
          text: 'Javelin',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: true,
          light: false,
          twohanded: false,
          weight: 2
        },
        {
          id: 5,
          text: 'Light Hammer',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d4',
          weapon: true,
          melee: true,
          thrown: true,
          light: true,
          twohanded: false,
          weight: 2
        },
        {
          id: 6,
          text: 'Mace',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 4
        },
        {
          id: 7,
          text: 'Quarterstaff',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 4
        },
        {
          id: 8,
          text: 'Sickle',
          type: 0,
          weaponCategory: 0,
          damage: 2,
          dice: '1d4',
          weapon: true,
          melee: true,
          thrown: false,
          light: true,
          twohanded: false,
          weight: 2
        },
        {
          id: 9,
          text: 'Spear',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 3
        },
        // Simple ranged
        {
          id: 10,
          text: 'Crossbow (Light)',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d8',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 5
        },
        {
          id: 11,
          text: 'Dart',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d4',
          weapon: true,
          melee: false,
          thrown: true,
          light: false,
          twohanded: false,
          weight: 0.25
        },
        {
          id: 12,
          text: 'Shortbow',
          type: 0,
          weaponCategory: 0,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 2
        },
        {
          id: 13,
          text: 'Sling',
          type: 0,
          weaponCategory: 0,
          damage: 0,
          dice: '1d4',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 0
        },
        // Martial melee
        {
          id: 14,
          text: 'Battleaxe',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 4
        },
        {
          id: 15,
          text: 'Flail',
          type: 0,
          weaponCategory: 1,
          damage: 0,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 2
        },
        {
          id: 16,
          text: 'Glaive',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d10',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 6
        },
        {
          id: 17,
          text: 'Greataxe',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d12',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 7
        },
        {
          id: 18,
          text: 'Greatsword',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '2d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 6
        },
        {
          id: 19,
          text: 'Halberd',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d10',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 6
        },
        {
          id: 20,
          text: 'Lance',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d12',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 6
        },
        {
          id: 21,
          text: 'Longsword',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 3
        },
        {
          id: 22,
          text: 'Maul',
          type: 0,
          weaponCategory: 1,
          damage: 0,
          dice: '2d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 10
        },
        {
          id: 23,
          text: 'Morningstar',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 4
        },
        {
          id: 24,
          text: 'Pike',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d10',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 18
        },
        {
          id: 25,
          text: 'Rapier',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 2
        },
        {
          id: 26,
          text: 'Scimitar',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: true,
          twohanded: false,
          weight: 3
        },
        {
          id: 27,
          text: 'Shortsword',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: false,
          light: true,
          twohanded: false,
          weight: 2
        },
        {
          id: 28,
          text: 'Trident',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: true,
          thrown: true,
          light: false,
          twohanded: false,
          weight: 4
        },
        {
          id: 29,
          text: 'War Pick',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 2
        },
        {
          id: 30,
          text: 'Warhammer',
          type: 0,
          weaponCategory: 1,
          damage: 0,
          dice: '1d8',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 2
        },
        {
          id: 31,
          text: 'Whip',
          type: 0,
          weaponCategory: 1,
          damage: 2,
          dice: '1d4',
          weapon: true,
          melee: true,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 3
        },
        // Martial ranged
        {
          id: 32,
          text: 'Blowgun',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: false,
          weight: 1
        },
        {
          id: 33,
          text: 'Crossbow (Hand)',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d6',
          weapon: true,
          melee: false,
          thrown: false,
          light: true,
          twohanded: false,
          weight: 3
        },
        {
          id: 34,
          text: 'Crossbow (Heavy)',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d10',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 18
        },
        {
          id: 35,
          text: 'Longbow',
          type: 0,
          weaponCategory: 1,
          damage: 1,
          dice: '1d8',
          weapon: true,
          melee: false,
          thrown: false,
          light: false,
          twohanded: true,
          weight: 2
        },
        {
          id: 36,
          text: 'Net',
          type: 0,
          weaponCategory: 1,
          damage: 0,
          dice: '0',
          weapon: true,
          melee: false,
          thrown: true,
          light: false,
          twohanded: false,
          weight: 3
        },
        // Items (TODO: Weights)
        // Packs
        {
          id: 37,
          type: 2,
          text: 'Explorer\'s pack',
          weapon: false,
        },
        {
          id: 38,
          type: 2,
          text: 'Diplomat\'s pack',
          weapon: false,
        },
        {
          id: 39,
          type: 2,
          text: 'Entertainer\'s pack',
          weapon: false,
        },
        {
          id: 40,
          type: 2,
          text: 'Priest\'s pack',
          weapon: false,
        },
        {
          id: 41,
          type: 2,
          text: 'Dungeoneer\'s pack',
          weapon: false,
        },
        {
          id: 42,
          type: 2,
          text: 'Burglar\'s pack',
          weapon: false,
        },
        {
          id: 43,
          type: 2,
          text: 'Scholar\'s pack',
          weapon: false,
        },
        // Armor
        {
          id: 44,
          type: 1,
          text: 'Padded armor',
          weapon: false,
        },
        {
          id: 45,
          type: 1,
          text: 'Leather armor',
          weapon: false,
        },
        {
          id: 46,
          type: 1,
          text: 'Studded leather armor',
          weapon: false,
        },
        {
          id: 47,
          type: 1,
          text: 'Hide armor',
          weapon: false,
        },
        {
          id: 48,
          type: 1,
          text: 'Chain shirt',
          weapon: false,
        },
        {
          id: 49,
          type: 1,
          text: 'Scale mail',
          weapon: false,
        },
        {
          id: 50,
          type: 1,
          text: 'Breastplate',
          weapon: false,
        },
        {
          id: 51,
          type: 1,
          text: 'Half plate',
          weapon: false,
        },
        {
          id: 52,
          type: 1,
          text: 'Ring mail',
          weapon: false,
        },
        {
          id: 53,
          type: 1,
          text: 'Chain mail',
          weapon: false,
        },
        {
          id: 54,
          type: 1,
          text: 'Splint',
          weapon: false,
        },
        {
          id: 55,
          type: 1,
          text: 'Plate',
          weapon: false,
        },
        // Shields
        {
          id: 56,
          type: 3,
          text: 'Wooden shield',
          weapon: false,
        },
        {
          id: 57,
          type: 3,
          text: 'Shield',
          weapon: false,
        },
        // Ammo
        {
          id: 58,
          type: 4,
          text: 'Arrows',
          weapon: false,
        },
        {
          id: 59,
          type: 4,
          text: 'Bolts',
          weapon: false,
        },
        {
          id: 60,
          type: 4,
          text: 'Sling Bullets',
          weapon: false,
        },
        {
          id: 61,
          type: 4,
          text: 'Blowgun Needles',
          weapon: false,
        },
        // Musical instruments
        {
          id: 62,
          type: 6,
          text: 'Lute',
          weapon: false,
        },
        {
          id: 63,
          type: 6,
          text: 'Musical instrument (________)',
          weapon: false,
        },
        // Other
        {
          id: 64,
          type: 6,
          text: 'Component pouch',
          weapon: false,
        },
        {
          id: 65,
          type: 6,
          text: 'Druidic focus',
          weapon: false,
        },
        // Adventurers equipment
        {
          id: 66,
          type: 6,
          text: 'Holy symbol',  /* A gift to you when you entered the priesthood */
          weapon: false,
        },
        {
          id: 67,
          type: 6,
          text: 'Prayer book/prayer wheel',
          weapon: false,
        },
        {
          id: 68,
          type: 6,
          text: 'Incense stick (5)',
          weapon: false,
        },
        {
          id: 69,
          type: 6,
          text: 'Vestments',
          weapon: false,
        },
        {
          id: 70,
          type: 6,
          text: 'Common clothes',
          weapon: false,
        },
        {
          id: 71,
          type: 6,
          text: 'Belt pouch',
          weapon: false,
        },
        {
          id: 72,
          type: 6,
          text: 'Fine clothes',
          weapon: false,
        },
        {
          id: 73,
          type: 6,
          text: 'Disguise kit',
          weapon: false,
        },
        {
          id: 74,
          type: 6,
          text: 'Con tools',  /* Tools of the con of your choice (ten stoppered bottles filled with colored liquid, a set of weighted dice, a deck of marked cards, or a signet ring of an imaginary duke) */
          weapon: false,
        },
        {
          id: 75,
          type: 6,
          text: 'Crowbar',
          weapon: false,
        },
        {
          id: 76,
          type: 6,
          text: 'Dark common clothes (w/ hood)',
          weapon: false,
        },
        {
          id: 77,
          type: 6,
          text: 'Admirer\'s favor', /* Love letter, lock of hair, or trinket */
          weapon: false,
        },
        {
          id: 78,
          type: 6,
          text: 'Costume',
          weapon: false,
        },
        {
          id: 79,
          type: 6,
          text: 'Artisan\'s tools (________)',
          weapon: false,
        },
        {
          id: 80,
          type: 6,
          text: 'Shovel',
          weapon: false,
        },
        {
          id: 81,
          type: 6,
          text: 'Iron pot',
          weapon: false,
        },
        {
          id: 82,
          type: 6,
          text: 'Guild introduction letter',
          weapon: false,
        },
        {
          id: 83,
          type: 6,
          text: 'Traveler\'s clothes',
          weapon: false,
        },
        {
          id: 84,
          type: 6,
          text: 'Scroll case of notes from studies or prayers',
          weapon: false,
        },
        {
          id: 85,
          type: 6,
          text: 'Winter blanket',
          weapon: false,
        },
        {
          id: 86,
          type: 6,
          text: 'Herbalism kit',
          weapon: false,
        },
        {
          id: 87,
          type: 6,
          text: 'Signet ring',
          weapon: false,
        },
        {
          id: 88,
          type: 6,
          text: 'Scroll of pedigree',
          weapon: false,
        },
        {
          id: 89,
          type: 6,
          text: 'Purse',
          weapon: false,
        },
        {
          id: 90,
          type: 6,
          text: 'Staff',
          weapon: false,
        },
        {
          id: 91,
          type: 6,
          text: 'Hunting trap',
          weapon: false,
        },
        {
          id: 92,
          type: 6,
          text: 'Animal trophy',
          weapon: false,
        },
        {
          id: 93,
          type: 6,
          text: 'Bottle of black ink',
          weapon: false,
        },
        {
          id: 94,
          type: 6,
          text: 'Quill',
          weapon: false,
        },
        {
          id: 95,
          type: 6,
          text: 'Small knife',
          weapon: false,
        },
        {
          id: 96,
          type: 6,
          text: 'Letter from a dead colleague posing a question you are unable to answer',
          weapon: false,
        },
        {
          id: 97,
          type: 6,
          text: 'Belaying pin (club)',
          weapon: false,
        },
        {
          id: 98,
          type: 6,
          text: '50 feet of silk rope',
          weapon: false,
        },
        {
          id: 99,
          type: 6,
          text: 'Lucky charm', /* Such as a rabbit foot or small stone with a hole in the center (or you may roll for a random trinket on the Trinkets table in chapter 5) */
          weapon: false,
        },
        {
          id: 100,
          type: 6,
          text: 'Insignia of rank',
          weapon: false,
        },
        {
          id: 101,
          type: 6,
          text: 'Trophy from a fallen enemy', /* A dagger, broken blade, or piece of a banner */
          weapon: false,
        },
        {
          id: 102,
          type: 6,
          text: 'Set of bone dice or deck of cards',
          weapon: false,
        },
        {
          id: 103,
          type: 6,
          text: 'Map of your home city',
          weapon: false,
        },
        {
          id: 104,
          type: 6,
          text: 'Pet mouse', /* Go for the eyes, boo! */
          weapon: false,
        },
        {
          id: 105,
          type: 6,
          text: 'A token to remember your parents by',
          weapon: false,
        },
        {
          id: 106,
          type: 6,
          text: 'Thieves\' tools',
          weapon: false,
        },
        {
          id: 107,
          type: 6,
          text: 'Arcane focus',
          weapon: false,
        },
        {
          id: 108,
          type: 6,
          text: 'Spellbook',
          weapon: false,
        },
      ],
      backgroundEquipment: [
        { id: 0, equipids: [66, 67, 68, 69, 70, 71] },
        { id: 1, equipids: [72, 73, 74, 71] },
        { id: 2, equipids: [75, 76, 71] },
        { id: 3, equipids: [63, 77, 78, 71] },
        { id: 4, equipids: [79, 80, 81, 70, 71] },
        { id: 5, equipids: [79, 82, 83, 71] },
        { id: 6, equipids: [84, 85, 70, 86] },
        { id: 7, equipids: [72, 87, 88, 89] },
        { id: 8, equipids: [90, 91, 92, 83, 71] },
        { id: 9, equipids: [93, 94, 95, 96, 70, 71] },
        { id: 10, equipids: [97, 98, 99, 70, 71] },
        { id: 11, equipids: [100, 101, 102, 70, 71] },
        { id: 12, equipids: [95, 103, 104, 105, 70, 71] },
      ],
      backgrounds: [
        { id: 0, text: 'Acolyte' },
        { id: 1, text: 'Charlatan' },
        { id: 2, text: 'Criminal' },
        { id: 3, text: 'Entertainer' },
        { id: 4, text: 'Folk Hero' },
        { id: 5, text: 'Guild Artisan' },
        { id: 6, text: 'Hermit' },
        { id: 7, text: 'Noble' },
        { id: 8, text: 'Outlander' },
        { id: 9, text: 'Sage' },
        { id: 10, text: 'Sailor' },
        { id: 11, text: 'Soldier' },
        { id: 12, text: 'Urchin' },
      ],
      backgroundCurrency: [
        { id: 0, currency: [0, 0, 0, 15, 0] }, /* CP, SP, EP, GP, PP */
        { id: 1, currency: [0, 0, 0, 15, 0] },
        { id: 2, currency: [0, 0, 0, 15, 0] },
        { id: 3, currency: [0, 0, 0, 15, 0] },
        { id: 4, currency: [0, 0, 0, 10, 0] },
        { id: 5, currency: [0, 0, 0, 15, 0] },
        { id: 6, currency: [0, 0, 0, 5, 0] },
        { id: 7, currency: [0, 0, 0, 25, 0] },
        { id: 8, currency: [0, 0, 0, 10, 0] },
        { id: 9, currency: [0, 0, 0, 10, 0] },
        { id: 10, currency: [0, 0, 0, 10, 0] },
        { id: 11, currency: [0, 0, 0, 10, 0] },
        { id: 12, currency: [0, 0, 0, 10, 0] },
      ],
      backgroundLangauges: [
        { id: 0, num: 2 },
        { id: 5, num: 1 },
        { id: 6, num: 1 },
        { id: 7, num: 1 },
        { id: 8, num: 1 },
        { id: 9, num: 2 },
      ],
      toolProficiencies: [
        { id: 0, text: 'Disguise kit' }, /* Start background tools */
        { id: 1, text: 'Forgery kit' },
        { id: 2, text: 'Gaming set (________)' },
        { id: 3, text: 'Thieves\' tools' },
        { id: 4, text: 'Musical instrument  (________)' },
        { id: 5, text: 'Artisan\'s tools (________)' },
        { id: 6, text: 'Vehicles (land)' },
        { id: 7, text: 'Herbalism kit' },
        { id: 8, text: 'Navigator\'s tools' },
        { id: 9, text: 'Vehicles (water)' },
      ],
      backgroundToolProficiencies: [
        { id: 0, profs: [] },
        { id: 1, profs: [0, 1] },
        { id: 2, profs: [2, 3] },
        { id: 3, profs: [0, 4] },
        { id: 4, profs: [5, 6] },
        { id: 5, profs: [5] },
        { id: 6, profs: [7] },
        { id: 7, profs: [2] },
        { id: 8, profs: [4] },
        { id: 9, profs: [] },
        { id: 10, profs: [8, 9] },
        { id: 11, profs: [2, 6] },
        { id: 12, profs: [0, 3] },
      ],
      personalityTraits: [ // http://www.enworld.org/forum/showthread.php?469002-List-of-All-Personality-Traits-Ideals-Bonds-amp-Flaws
        { id: 0, roll: 1, bg: 0, text: '1. I idolize (p.127)' },
        { id: 1, roll: 2, bg: 0, text: '2. I can (p.127)' },
        { id: 2, roll: 3, bg: 0, text: '3. I see (p.127)' },
        { id: 3, roll: 4, bg: 0, text: '4. Nothing can (p.127)' },
        { id: 4, roll: 5, bg: 0, text: '5. I quote (p.127)' },
        { id: 5, roll: 6, bg: 0, text: '6. I am (p.127)' },
        { id: 6, roll: 7, bg: 0, text: '7. I\'ve enjoyed (p.127)' },
        { id: 7, roll: 8, bg: 0, text: '8. I\'ve spent (p.127)' },
        { id: 8, roll: 1, bg: 1, text: '1. I fall (p.128)' },
        { id: 9, roll: 2, bg: 1, text: '2. I have (p.128)' },
        { id: 10, roll: 3, bg: 1, text: '3. Flattery is (p.128)' },
        { id: 11, roll: 4, bg: 1, text: '4. I\'m a (p.128)' },
        { id: 12, roll: 5, bg: 1, text: '5. I lie (p.128)' },
        { id: 13, roll: 6, bg: 1, text: '6. Sarcasm and (p.128)' },
        { id: 14, roll: 7, bg: 1, text: '7. I keep (p.128)' },
        { id: 15, roll: 8, bg: 1, text: '8. I pocket (p.128)' },
        { id: 16, roll: 1, bg: 2, text: '1. I always (p.129)' },
        { id: 17, roll: 2, bg: 2, text: '2. I am (p.129)' },
        { id: 18, roll: 3, bg: 2, text: '3. The first (p.129)' },
        { id: 19, roll: 4, bg: 2, text: '4. I would (p.129)' },
        { id: 20, roll: 5, bg: 2, text: '5. I am (p.129)' },
        { id: 21, roll: 6, bg: 2, text: '6. I don\'t (p.129)' },
        { id: 22, roll: 7, bg: 2, text: '7. The best (p.129)' },
        { id: 23, roll: 8, bg: 2, text: '8. I blow (p.129)' },
        { id: 24, roll: 1, bg: 3, text: '1. I know (p.130)' },
        { id: 25, roll: 2, bg: 3, text: '2. Whenever I (p.130)' },
        { id: 26, roll: 3, bg: 3, text: '3. I\'m a (p.130)' },
        { id: 27, roll: 4, bg: 3, text: '4. Nobody stays (p.130)' },
        { id: 28, roll: 5, bg: 3, text: '5. I love (p.130)' },
        { id: 29, roll: 6, bg: 3, text: '6. I get (p.130)' },
        { id: 30, roll: 7, bg: 3, text: '7. I\'ll settle (p.130)' },
        { id: 31, roll: 8, bg: 3, text: '8. I change (p.130)' },
        { id: 32, roll: 1, bg: 4, text: '1. I judge (p.131)' },
        { id: 33, roll: 2, bg: 4, text: '2. If someone (p.131)' },
        { id: 34, roll: 3, bg: 4, text: '3. When I (p.131)' },
        { id: 35, roll: 4, bg: 4, text: '4. I have (p.131)' },
        { id: 36, roll: 5, bg: 4, text: '5. I\'m confident (p.131)' },
        { id: 37, roll: 6, bg: 4, text: '6. Thinking is (p.131)' },
        { id: 38, roll: 7, bg: 4, text: '7. I misuse (p.131)' },
        { id: 39, roll: 8, bg: 4, text: '8. I get (p.131)' },
        { id: 40, roll: 1, bg: 5, text: '1. I believe (p.133)' },
        { id: 41, roll: 2, bg: 5, text: '2. I\'m a (p.133)' },
        { id: 42, roll: 3, bg: 5, text: '3. I always (p.133)' },
        { id: 43, roll: 4, bg: 5, text: '4. I\'m full (p.133)' },
        { id: 44, roll: 5, bg: 5, text: '5. I\'m rude (p.133)' },
        { id: 45, roll: 6, bg: 5, text: '6. I like (p.133)' },
        { id: 46, roll: 7, bg: 5, text: '7. I don\'t (p.133)' },
        { id: 47, roll: 8, bg: 5, text: '8. I don\'t (p.133)' },
        { id: 48, roll: 1, bg: 6, text: '1. I\'ve been (p.134)' },
        { id: 49, roll: 2, bg: 6, text: '2. I am (p.134)' },
        { id: 50, roll: 3, bg: 6, text: '3. The leader (p.134)' },
        { id: 51, roll: 4, bg: 6, text: '4. I feel (p.134)' },
        { id: 52, roll: 5, bg: 6, text: '5. I\'m oblivious (p.134)' },
        { id: 53, roll: 6, bg: 6, text: '6. I connect (p.134)' },
        { id: 54, roll: 7, bg: 6, text: '7. I often (p.134)' },
        { id: 55, roll: 8, bg: 6, text: '8. I often (p.134)' },
        { id: 56, roll: 1, bg: 7, text: '1. My eloquent (p.135)' },
        { id: 57, roll: 2, bg: 7, text: '2. The common (p.135)' },
        { id: 58, roll: 3, bg: 7, text: '3. No one (p.135)' },
        { id: 59, roll: 4, bg: 7, text: '4. I take (p.135)' },
        { id: 60, roll: 5, bg: 7, text: '5. I don\'t (p.135)' },
        { id: 61, roll: 6, bg: 7, text: '6. Despite my (p.135)' },
        { id: 62, roll: 7, bg: 7, text: '7. My favor (p.135)' },
        { id: 63, roll: 8, bg: 7, text: '8. If you (p.135)' },
        { id: 64, roll: 1, bg: 8, text: '1. I\'m driven (p.137)' },
        { id: 65, roll: 2, bg: 8, text: '2. I watch (p.137)' },
        { id: 66, roll: 3, bg: 8, text: '3. I once (p.137)' },
        { id: 67, roll: 4, bg: 8, text: '4. I have (p.137)' },
        { id: 68, roll: 5, bg: 8, text: '5. I place (p.137)' },
        { id: 69, roll: 6, bg: 8, text: '6. I\'m always (p.137)' },
        { id: 70, roll: 7, bg: 8, text: '7. I feel (p.137)' },
        { id: 71, roll: 8, bg: 8, text: '8. I was (p.137)' },
        { id: 72, roll: 1, bg: 9, text: '1. I use (p.138)' },
        { id: 73, roll: 2, bg: 9, text: '2. I\'ve read (p.138)' },
        { id: 74, roll: 3, bg: 9, text: '3. I\'m used (p.138)' },
        { id: 75, roll: 4, bg: 9, text: '4. There\'s nothing (p.138)' },
        { id: 76, roll: 5, bg: 9, text: '5. I\'m willing (p.138)' },
        { id: 77, roll: 6, bg: 9, text: '6. I...speak (p.138)' },
        { id: 78, roll: 7, bg: 9, text: '7. I am (p.138)' },
        { id: 79, roll: 8, bg: 9, text: '8. I\'m convinced (p.138)' },
        { id: 80, roll: 1, bg: 10, text: '1. My friends (p.139)' },
        { id: 81, roll: 2, bg: 10, text: '2. I work (p.139)' },
        { id: 82, roll: 3, bg: 10, text: '3. I enjoy (p.139)' },
        { id: 83, roll: 4, bg: 10, text: '4. I stretch (p.139)' },
        { id: 84, roll: 5, bg: 10, text: '5. To me (p.139)' },
        { id: 85, roll: 6, bg: 10, text: '6. I never (p.139)' },
        { id: 86, roll: 7, bg: 10, text: '7. My language (p.139)' },
        { id: 87, roll: 8, bg: 10, text: '8. I like (p.139)' },
        { id: 88, roll: 1, bg: 11, text: '1. I\'m always (p.140)' },
        { id: 89, roll: 2, bg: 11, text: '2. I\'m haunted (p.140)' },
        { id: 90, roll: 3, bg: 11, text: '3. I\'ve lost (p.140)' },
        { id: 91, roll: 4, bg: 11, text: '4. I\'m full (p.140)' },
        { id: 92, roll: 5, bg: 11, text: '5. I can (p.140)' },
        { id: 93, roll: 6, bg: 11, text: '6. I enjoy (p.140)' },
        { id: 94, roll: 7, bg: 11, text: '7. I have (p.140)' },
        { id: 95, roll: 8, bg: 11, text: '8. I face (p.140)' },
        { id: 96, roll: 1, bg: 12, text: '1. I hide (p.141)' },
        { id: 97, roll: 2, bg: 12, text: '2. I ask (p.141)' },
        { id: 98, roll: 3, bg: 12, text: '3. I like (p.141)' },
        { id: 99, roll: 4, bg: 12, text: '4. I sleep (p.141)' },
        { id: 100, roll: 5, bg: 12, text: '5. I eat (p.141)' },
        { id: 101, roll: 6, bg: 12, text: '6. I think (p.141)' },
        { id: 102, roll: 7, bg: 12, text: '7. I don\'t (p.141)' },
        { id: 103, roll: 8, bg: 12, text: '8. I bluntly (p.141)' },
      ],
      ideals: [
        { id: 0, bg: 0, text: '1. Tradition (p.127)' },
        { id: 1, bg: 0, text: '2. Charity (p.127)' },
        { id: 2, bg: 0, text: '3. Change (p.127)' },
        { id: 3, bg: 0, text: '4. Power (p.127)' },
        { id: 4, bg: 0, text: '5. Faith (p.127)' },
        { id: 5, bg: 0, text: '6. Aspiration (p.127)' },
        { id: 6, bg: 1, text: '1. Independence (p.128)' },
        { id: 7, bg: 1, text: '2. Fairness (p.128)' },
        { id: 8, bg: 1, text: '3. Charity (p.128)' },
        { id: 9, bg: 1, text: '4. Creativity (p.128)' },
        { id: 10, bg: 1, text: '5. Friendship (p.128)' },
        { id: 11, bg: 1, text: '6. Aspiration (p.128)' },
        { id: 12, bg: 2, text: '1. Honor (p.129)' },
        { id: 13, bg: 2, text: '2. Freedom (p.129)' },
        { id: 14, bg: 2, text: '3. Charity (p.129)' },
        { id: 15, bg: 2, text: '4. Greed (p.129)' },
        { id: 16, bg: 2, text: '5. People (p.129)' },
        { id: 17, bg: 2, text: '6. Redemption (p.129)' },
        { id: 18, bg: 3, text: '1. Beauty (p.131)' },
        { id: 19, bg: 3, text: '2. Tradition (p.131)' },
        { id: 20, bg: 3, text: '3. Creativity (p.131)' },
        { id: 21, bg: 3, text: '4. Greed (p.131)' },
        { id: 22, bg: 3, text: '5. People (p.131)' },
        { id: 23, bg: 3, text: '6. Honesty (p.131)' },
        { id: 24, bg: 4, text: '1. Respect (p.132)' },
        { id: 25, bg: 4, text: '2. Fairness (p.132)' },
        { id: 26, bg: 4, text: '3. Freedom (p.132)' },
        { id: 27, bg: 4, text: '4. Might (p.132)' },
        { id: 28, bg: 4, text: '5. Sincerity (p.132)' },
        { id: 29, bg: 4, text: '6. Destiny (p.132)' },
        { id: 30, bg: 5, text: '1. Community (p.133)' },
        { id: 31, bg: 5, text: '2. Generosity (p.133)' },
        { id: 32, bg: 5, text: '3. Freedom (p.133)' },
        { id: 33, bg: 5, text: '4. Greed (p.133)' },
        { id: 34, bg: 5, text: '5. People (p.133)' },
        { id: 35, bg: 5, text: '6. Aspiration (p.133)' },
        { id: 36, bg: 6, text: '1. Greater Good (p.134)' },
        { id: 37, bg: 6, text: '2. Logic (p.134)' },
        { id: 38, bg: 6, text: '3. Free Thinking (p.134)' },
        { id: 39, bg: 6, text: '4. Power (p.134)' },
        { id: 40, bg: 6, text: '5. Live and Let Live (p.134)' },
        { id: 41, bg: 6, text: '6. Self-Knowledge (p.134)' },
        { id: 42, bg: 7, text: '1. Respect (p.136)' },
        { id: 43, bg: 7, text: '2. Responsibility (p.136)' },
        { id: 44, bg: 7, text: '3. Independence (p.136)' },
        { id: 45, bg: 7, text: '4. Power (p.136)' },
        { id: 46, bg: 7, text: '5. Family (p.136)' },
        { id: 47, bg: 7, text: '6. Noble Obligation (p.136)' },
        { id: 48, bg: 8, text: '1. Change (p.137)' },
        { id: 49, bg: 8, text: '2. Greater Good (p.137)' },
        { id: 50, bg: 8, text: '3. Honor (p.137)' },
        { id: 51, bg: 8, text: '4. Might (p.137)' },
        { id: 52, bg: 8, text: '5. Nature (p.137)' },
        { id: 53, bg: 8, text: '6. Glory (p.137)' },
        { id: 54, bg: 9, text: '1. Knowledge (p.138)' },
        { id: 55, bg: 9, text: '2. Beauty (p.138)' },
        { id: 56, bg: 9, text: '3. Logic (p.138)' },
        { id: 57, bg: 9, text: '4. No Limits (p.138)' },
        { id: 58, bg: 9, text: '5. Power (p.138)' },
        { id: 59, bg: 9, text: '6. Self-Improvement (p.138)' },
        { id: 60, bg: 10, text: '1. Respect (p.139)' },
        { id: 61, bg: 10, text: '2. Fairness (p.139)' },
        { id: 62, bg: 10, text: '3. Freedom (p.139)' },
        { id: 63, bg: 10, text: '4. Mastery (p.139)' },
        { id: 64, bg: 10, text: '5. People (p.139)' },
        { id: 65, bg: 10, text: '6. Aspiration (p.139)' },
        { id: 66, bg: 11, text: '1. Greater Good (p.140)' },
        { id: 67, bg: 11, text: '2. Responsibility (p.140)' },
        { id: 68, bg: 11, text: '3. Independence (p.140)' },
        { id: 69, bg: 11, text: '4. Might (p.140)' },
        { id: 70, bg: 11, text: '5. Live and Let Live (p.140)' },
        { id: 71, bg: 11, text: '6. Nation (p.140)' },
        { id: 72, bg: 12, text: '1. Respect (p.141)' },
        { id: 73, bg: 12, text: '2. Community (p.141)' },
        { id: 74, bg: 12, text: '3. Change (p.141)' },
        { id: 75, bg: 12, text: '4. Retribution (p.141)' },
        { id: 76, bg: 12, text: '5. People (p.141)' },
        { id: 77, bg: 12, text: '6. Aspiration (p.141)' },
      ],
      bonds: [
        { id: 0, bg: 0, text: '1. (p.127)' },
        { id: 1, bg: 0, text: '2. (p.127)' },
        { id: 2, bg: 0, text: '3. (p.127)' },
        { id: 3, bg: 0, text: '4. (p.127)' },
        { id: 4, bg: 0, text: '5. (p.127)' },
        { id: 5, bg: 0, text: '6. (p.127)' },
        { id: 6, bg: 1, text: '1. (p.128)' },
        { id: 7, bg: 1, text: '2. (p.128)' },
        { id: 8, bg: 1, text: '3. (p.128)' },
        { id: 9, bg: 1, text: '4. (p.128)' },
        { id: 10, bg: 1, text: '5. (p.128)' },
        { id: 11, bg: 1, text: '6. (p.128)' },
        { id: 12, bg: 2, text: '1. (p.129)' },
        { id: 13, bg: 2, text: '2. (p.129)' },
        { id: 14, bg: 2, text: '3. (p.129)' },
        { id: 15, bg: 2, text: '4. (p.129)' },
        { id: 16, bg: 2, text: '5. (p.129)' },
        { id: 17, bg: 2, text: '6. (p.129)' },
        { id: 18, bg: 3, text: '1. (p.131)' },
        { id: 19, bg: 3, text: '2. (p.131)' },
        { id: 20, bg: 3, text: '3. (p.131)' },
        { id: 21, bg: 3, text: '4. (p.131)' },
        { id: 22, bg: 3, text: '5. (p.131)' },
        { id: 23, bg: 3, text: '6. (p.131)' },
        { id: 24, bg: 4, text: '1. (p.132)' },
        { id: 25, bg: 4, text: '2. (p.132)' },
        { id: 26, bg: 4, text: '3. (p.132)' },
        { id: 27, bg: 4, text: '4. (p.132)' },
        { id: 28, bg: 4, text: '5. (p.132)' },
        { id: 29, bg: 4, text: '6. (p.132)' },
        { id: 30, bg: 5, text: '1. (p.133)' },
        { id: 31, bg: 5, text: '2. (p.133)' },
        { id: 32, bg: 5, text: '3. (p.133)' },
        { id: 33, bg: 5, text: '4. (p.133)' },
        { id: 34, bg: 5, text: '5. (p.133)' },
        { id: 35, bg: 5, text: '6. (p.133)' },
        { id: 36, bg: 6, text: '1. (p.135)' },
        { id: 37, bg: 6, text: '2. (p.135)' },
        { id: 38, bg: 6, text: '3. (p.135)' },
        { id: 39, bg: 6, text: '4. (p.135)' },
        { id: 40, bg: 6, text: '5. (p.135)' },
        { id: 41, bg: 6, text: '6. (p.135)' },
        { id: 42, bg: 7, text: '1. (p.136)' },
        { id: 43, bg: 7, text: '2. (p.136)' },
        { id: 44, bg: 7, text: '3. (p.136)' },
        { id: 45, bg: 7, text: '4. (p.136)' },
        { id: 46, bg: 7, text: '5. (p.136)' },
        { id: 47, bg: 7, text: '6. (p.136)' },
        { id: 48, bg: 8, text: '1. (p.137)' },
        { id: 49, bg: 8, text: '2. (p.137)' },
        { id: 50, bg: 8, text: '3. (p.137)' },
        { id: 51, bg: 8, text: '4. (p.137)' },
        { id: 52, bg: 8, text: '5. (p.137)' },
        { id: 53, bg: 8, text: '6. (p.137)' },
        { id: 54, bg: 9, text: '1. (p.138)' },
        { id: 55, bg: 9, text: '2. (p.138)' },
        { id: 56, bg: 9, text: '3. (p.138)' },
        { id: 57, bg: 9, text: '4. (p.138)' },
        { id: 58, bg: 9, text: '5. (p.138)' },
        { id: 59, bg: 9, text: '6. (p.138)' },
        { id: 60, bg: 10, text: '1. (p.139)' },
        { id: 61, bg: 10, text: '2. (p.139)' },
        { id: 62, bg: 10, text: '3. (p.139)' },
        { id: 63, bg: 10, text: '4. (p.139)' },
        { id: 64, bg: 10, text: '5. (p.139)' },
        { id: 65, bg: 10, text: '6. (p.139)' },
        { id: 66, bg: 11, text: '1. (p.141)' },
        { id: 67, bg: 11, text: '2. (p.141)' },
        { id: 68, bg: 11, text: '3. (p.141)' },
        { id: 69, bg: 11, text: '4. (p.141)' },
        { id: 70, bg: 11, text: '5. (p.141)' },
        { id: 71, bg: 11, text: '6. (p.141)' },
        { id: 72, bg: 12, text: '1. (p.141)' },
        { id: 73, bg: 12, text: '2. (p.141)' },
        { id: 74, bg: 12, text: '3. (p.141)' },
        { id: 75, bg: 12, text: '4. (p.141)' },
        { id: 76, bg: 12, text: '5. (p.141)' },
        { id: 77, bg: 12, text: '6. (p.141)' },
      ],
      flaws: [
        { id: 0, bg: 0, text: '1. (p.127)' },
        { id: 1, bg: 0, text: '2. (p.127)' },
        { id: 2, bg: 0, text: '3. (p.127)' },
        { id: 3, bg: 0, text: '4. (p.127)' },
        { id: 4, bg: 0, text: '5. (p.127)' },
        { id: 5, bg: 0, text: '6. (p.127)' },
        { id: 6, bg: 1, text: '1. (p.128)' },
        { id: 7, bg: 1, text: '2. (p.128)' },
        { id: 8, bg: 1, text: '3. (p.128)' },
        { id: 9, bg: 1, text: '4. (p.128)' },
        { id: 10, bg: 1, text: '5. (p.128)' },
        { id: 11, bg: 1, text: '6. (p.128)' },
        { id: 12, bg: 2, text: '1. (p.130)' },
        { id: 13, bg: 2, text: '2. (p.130)' },
        { id: 14, bg: 2, text: '3. (p.130)' },
        { id: 15, bg: 2, text: '4. (p.130)' },
        { id: 16, bg: 2, text: '5. (p.130)' },
        { id: 17, bg: 2, text: '6. (p.130)' },
        { id: 18, bg: 3, text: '1. (p.131)' },
        { id: 19, bg: 3, text: '2. (p.131)' },
        { id: 20, bg: 3, text: '3. (p.131)' },
        { id: 21, bg: 3, text: '4. (p.131)' },
        { id: 22, bg: 3, text: '5. (p.131)' },
        { id: 23, bg: 3, text: '6. (p.131)' },
        { id: 24, bg: 4, text: '1. (p.132)' },
        { id: 25, bg: 4, text: '2. (p.132)' },
        { id: 26, bg: 4, text: '3. (p.132)' },
        { id: 27, bg: 4, text: '4. (p.132)' },
        { id: 28, bg: 4, text: '5. (p.132)' },
        { id: 29, bg: 4, text: '6. (p.132)' },
        { id: 30, bg: 5, text: '1. (p.133)' },
        { id: 31, bg: 5, text: '2. (p.133)' },
        { id: 32, bg: 5, text: '3. (p.133)' },
        { id: 33, bg: 5, text: '4. (p.133)' },
        { id: 34, bg: 5, text: '5. (p.133)' },
        { id: 35, bg: 5, text: '6. (p.133)' },
        { id: 36, bg: 6, text: '1. (p.135)' },
        { id: 37, bg: 6, text: '2. (p.135)' },
        { id: 38, bg: 6, text: '3. (p.135)' },
        { id: 39, bg: 6, text: '4. (p.135)' },
        { id: 40, bg: 6, text: '5. (p.135)' },
        { id: 41, bg: 6, text: '6. (p.135)' },
        { id: 42, bg: 7, text: '1. (p.136)' },
        { id: 43, bg: 7, text: '2. (p.136)' },
        { id: 44, bg: 7, text: '3. (p.136)' },
        { id: 45, bg: 7, text: '4. (p.136)' },
        { id: 46, bg: 7, text: '5. (p.136)' },
        { id: 47, bg: 7, text: '6. (p.136)' },
        { id: 48, bg: 8, text: '1. (p.137)' },
        { id: 49, bg: 8, text: '2. (p.137)' },
        { id: 50, bg: 8, text: '3. (p.137)' },
        { id: 51, bg: 8, text: '4. (p.137)' },
        { id: 52, bg: 8, text: '5. (p.137)' },
        { id: 53, bg: 8, text: '6. (p.137)' },
        { id: 54, bg: 9, text: '1. (p.138)' },
        { id: 55, bg: 9, text: '2. (p.138)' },
        { id: 56, bg: 9, text: '3. (p.138)' },
        { id: 57, bg: 9, text: '4. (p.138)' },
        { id: 58, bg: 9, text: '5. (p.138)' },
        { id: 59, bg: 9, text: '6. (p.138)' },
        { id: 60, bg: 10, text: '1. (p.139)' },
        { id: 61, bg: 10, text: '2. (p.139)' },
        { id: 62, bg: 10, text: '3. (p.139)' },
        { id: 63, bg: 10, text: '4. (p.139)' },
        { id: 64, bg: 10, text: '5. (p.139)' },
        { id: 65, bg: 10, text: '6. (p.139)' },
        { id: 66, bg: 11, text: '1. (p.141)' },
        { id: 67, bg: 11, text: '2. (p.141)' },
        { id: 68, bg: 11, text: '3. (p.141)' },
        { id: 69, bg: 11, text: '4. (p.141)' },
        { id: 70, bg: 11, text: '5. (p.141)' },
        { id: 71, bg: 11, text: '6. (p.141)' },
        { id: 72, bg: 12, text: '1. (p.141)' },
        { id: 73, bg: 12, text: '2. (p.141)' },
        { id: 74, bg: 12, text: '3. (p.141)' },
        { id: 75, bg: 12, text: '4. (p.141)' },
        { id: 76, bg: 12, text: '5. (p.141)' },
        { id: 77, bg: 12, text: '6. (p.141)' },
      ],
      racialBonuses: [
        { id: 0, bonuses: [0, 0, 2, 0, 0, 0] },
        { id: 1, bonuses: [0, 0, 2, 0, 1, 0] },
        { id: 2, bonuses: [2, 0, 2, 0, 0, 0] },
        { id: 3, bonuses: [0, 2, 0, 0, 0, 0] },
        { id: 4, bonuses: [0, 2, 0, 1, 0, 0] },
        { id: 5, bonuses: [0, 2, 0, 0, 1, 0] },
        { id: 6, bonuses: [0, 2, 0, 0, 0, 1] },
        { id: 7, bonuses: [0, 2, 0, 0, 0, 0] },
        { id: 8, bonuses: [0, 2, 0, 0, 0, 1] },
        { id: 9, bonuses: [0, 2, 1, 0, 0, 0] },
        { id: 10, bonuses: [1, 1, 1, 1, 1, 1] },
        { id: 11, bonuses: [2, 0, 0, 0, 0, 1] },
        { id: 12, bonuses: [0, 0, 0, 2, 0, 0] },
        { id: 13, bonuses: [0, 1, 0, 2, 0, 0] },
        { id: 14, bonuses: [0, 0, 1, 2, 0, 0] },
        { id: 15, bonuses: [0, 0, 0, 0, 0, 2] },
        { id: 16, bonuses: [2, 0, 1, 0, 0, 0] },
        { id: 17, bonuses: [0, 0, 0, 1, 0, 2] },
      ],
      speeds: [ /* By race */
        { id: 0, value: 25 },
        { id: 1, value: 25 },
        { id: 2, value: 25 },
        { id: 3, value: 30 },
        { id: 4, value: 30 },
        { id: 5, value: 35 },
        { id: 6, value: 30 },
        { id: 7, value: 25 },
        { id: 8, value: 25 },
        { id: 9, value: 25 },
        { id: 10, value: 30 },
        { id: 11, value: 30 },
        { id: 12, value: 25 },
        { id: 13, value: 25 },
        { id: 14, value: 25 },
        { id: 15, value: 30 },
        { id: 16, value: 30 },
        { id: 17, value: 30 },
      ],
      raceLanguages: [
        { id: 0, languages: [0, 1] },
        { id: 1, languages: [0, 2] },
        { id: 2, languages: [0, 6] },
        { id: 3, languages: [0] },
        { id: 4, languages: [0, 10] },
        { id: 5, languages: [0, 4] },
        { id: 6, languages: [0, 2] },
        { id: 7, languages: [0, 7] },
        { id: 8, languages: [0, 12] },
      ],
      raceExtraLanguages: [
        { id: 0, extras: 0 },
        { id: 1, extras: 0 },
        { id: 2, extras: 0 },
        { id: 3, extras: 0 },
        { id: 4, extras: 1 },
        { id: 5, extras: 0 },
        { id: 6, extras: 0 },
        { id: 7, extras: 0 },
        { id: 8, extras: 0 },
        { id: 9, extras: 0 },
        { id: 10, extras: 1 },
        { id: 11, extras: 0 },
        { id: 12, extras: 0 },
        { id: 13, extras: 0 },
        { id: 14, extras: 0 },
        { id: 15, extras: 1 },
        { id: 16, extras: 0 },
        { id: 17, extras: 0 },
      ],
      languages: [
        { id: 0, text: 'Common' },
        { id: 1, text: 'Dwarvish' },
        { id: 2, text: 'Elvish' },
        { id: 3, text: 'Giant' },
        { id: 4, text: 'Gnomish' },
        { id: 5, text: 'Goblin' },
        { id: 6, text: 'Halfling' },
        { id: 7, text: 'Orc' },
        { id: 8, text: 'Abyssal' }, /* Exotic */
        { id: 9, text: 'Celestial' },
        { id: 10, text: 'Draconic' },
        { id: 11, text: 'Deep Speech' },
        { id: 12, text: 'Infernal' },
        { id: 13, text: 'Primordial' },
        { id: 14, text: 'Sylvan' },
        { id: 15, text: 'Undercommon' },
        { id: 16, text: 'Druidic' },
      ],
      proficiencyLevels: [
        { level: 1, bonus: 2 },
        { level: 2, bonus: 2 },
        { level: 3, bonus: 2 },
        { level: 4, bonus: 2 },
        { level: 5, bonus: 3 },
        { level: 6, bonus: 3 },
        { level: 7, bonus: 3 },
        { level: 8, bonus: 3 },
        { level: 9, bonus: 4 },
        { level: 10, bonus: 4 },
        { level: 11, bonus: 4 },
        { level: 12, bonus: 4 },
        { level: 13, bonus: 5 },
        { level: 14, bonus: 5 },
        { level: 15, bonus: 5 },
        { level: 16, bonus: 5 },
        { level: 17, bonus: 6 },
        { level: 18, bonus: 6 },
        { level: 19, bonus: 6 },
        { level: 20, bonus: 6 },
      ],
      features: [
        { id: 0, classid: 0, name: '' }
      ],
      savingThrows: [
        { id: 0, throws: [0, 2] },
        { id: 1, throws: [1, 5] },
        { id: 2, throws: [4, 5] },
        { id: 3, throws: [3, 4] },
        { id: 4, throws: [0, 2] },
        { id: 5, throws: [0, 1] },
        { id: 6, throws: [4, 5] },
        { id: 7, throws: [0, 1] },
        { id: 8, throws: [1, 3] },
        { id: 9, throws: [2, 5] },
        { id: 10, throws: [4, 5] },
        { id: 11, throws: [3, 4] },
      ],
      hitDice: [
        { id: 0, num: 1, dice: 12 },
        { id: 1, num: 1, dice: 8 },
        { id: 2, num: 1, dice: 8 },
        { id: 3, num: 1, dice: 8 },
        { id: 4, num: 1, dice: 10 },
        { id: 5, num: 1, dice: 8 },
        { id: 6, num: 1, dice: 10 },
        { id: 7, num: 1, dice: 10 },
        { id: 8, num: 1, dice: 8 },
        { id: 9, num: 1, dice: 6 },
        { id: 10, num: 1, dice: 8 },
        { id: 11, num: 1, dice: 6 },
      ],
      startingHp: [ /* +Constitution */
        { id: 0, hp: 12 },
        { id: 1, hp: 8 },
        { id: 2, hp: 8 },
        { id: 3, hp: 8 },
        { id: 4, hp: 10 },
        { id: 5, hp: 8 },
        { id: 6, hp: 10 },
        { id: 7, hp: 10 },
        { id: 8, hp: 8 },
        { id: 9, hp: 6 },
        { id: 10, hp: 8 },
        { id: 11, hp: 6 },
      ],
      alignmentLawChaos: [
        { id: 0, text: 'Lawful' },
        { id: 1, text: 'Neutral' },
        { id: 2, text: 'Chaotic' },
      ],
      alignmentGoodEvil: [
        { id: 0, text: 'Good' },
        { id: 1, text: 'Neutral' },
        { id: 2, text: 'Evil' },
      ],
      backgroundProficiencies: [
        { id: 0, profs: [6, 14] },
        { id: 1, profs: [4, 15] },
        { id: 2, profs: [4, 16] },
        { id: 3, profs: [0, 12] },
        { id: 4, profs: [1, 17] },
        { id: 5, profs: [6, 13] },
        { id: 6, profs: [9, 13] },
        { id: 7, profs: [5, 13] },
        { id: 8, profs: [3, 17] },
        { id: 9, profs: [2, 5] },
        { id: 10, profs: [3, 11] },
        { id: 11, profs: [3, 7] },
        { id: 12, profs: [15, 16] },
      ],
      classProficiencies: [
        { id: 0, num: 2, profs: [1, 3, 7, 10, 11, 17] },
        { id: 1, num: 3, profs: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17] },
        { id: 2, num: 2, profs: [5, 6, 9, 13, 14] },
        { id: 3, num: 2, profs: [1, 2, 6, 9, 10, 11, 17] },
        { id: 4, num: 2, profs: [0, 1, 3, 5, 6, 7, 11, 17] },
        { id: 5, num: 2, profs: [0, 3, 5, 6, 14, 16] },
        { id: 6, num: 2, profs: [3, 6, 7, 9, 13, 14] },
        { id: 7, num: 3, profs: [1, 3, 6, 8, 10, 11, 16, 17] },
        { id: 8, num: 4, profs: [0, 3, 4, 6, 7, 8, 11, 12, 13, 15, 16] },
        { id: 9, num: 2, profs: [2, 4, 6, 7, 13, 14] },
        { id: 10, num: 2, profs: [2, 4, 5, 7, 8, 10, 14] },
        { id: 11, num: 2, profs: [2, 5, 6, 8, 9, 14] },
      ],
      skills: [
        { id: 0, attr: 1, text: 'Acrobatics (Dex)' },
        { id: 1, attr: 4, text: 'Animal Handling (Wis)' },
        { id: 2, attr: 3, text: 'Arcana (Int)' },
        { id: 3, attr: 0, text: 'Athletics (Str)' },
        { id: 4, attr: 5, text: 'Deception (Cha)' },
        { id: 5, attr: 3, text: 'History (Int)' },
        { id: 6, attr: 4, text: 'Insight (Wis)' },
        { id: 7, attr: 5, text: 'Intimidation (Cha)' },
        { id: 8, attr: 3, text: 'Investigation (Int)' },
        { id: 9, attr: 4, text: 'Medicine (Wis)' },
        { id: 10, attr: 3, text: 'Nature (Int)' },
        { id: 11, attr: 4, text: 'Perception (Wis)' },
        { id: 12, attr: 5, text: 'Performance (Cha)' },
        { id: 13, attr: 5, text: 'Persuasion (Cha)' },
        { id: 14, attr: 3, text: 'Religion (Int)' },
        { id: 15, attr: 1, text: 'Sleight of Hand (Dex)' },
        { id: 16, attr: 1, text: 'Stealth (Dex)' },
        { id: 17, attr: 4, text: 'Survival (Wis)' },
      ],
      standardStatArray: [15, 14, 13, 12, 10, 8],
      statRolls: [15, 14, 13, 12, 10, 8],
      statBlocks: [
        { id: 0, text: 'Strength' },
        { id: 1, text: 'Dexterity' },
        { id: 2, text: 'Constitution' },
        { id: 3, text: 'Intelligence' },
        { id: 4, text: 'Wisdom' },
        { id: 5, text: 'Charisma' },
      ],
      statModifiers: [
        { val: 1, modifier: -5 },
        { val: 3, modifier: -4 },
        { val: 5, modifier: -3 },
        { val: 7, modifier: -2 },
        { val: 9, modifier: -1 },
        { val: 11, modifier: 0 },
        { val: 13, modifier: 1 },
        { val: 15, modifier: 2 },
        { val: 17, modifier: 3 },
        { val: 19, modifier: 4 },
        { val: 21, modifier: 5 },
        { val: 23, modifier: 6 },
        { val: 25, modifier: 7 },
        { val: 27, modifier: 8 },
        { val: 29, modifier: 9 },
        { val: 30, modifier: 10 },
      ],
      output: {
        level: 1,
        characterName: '',
        playerName: '',
        race: 'Dwarf',
        raceid: 0,
        class: 'Barbarian',
        classid: 0,
        alignmentLawChaos: 'Lawful',
        alignmentGoodEvil: 'Good',
        background: 'Acolyte',
        backgroundid: 0,
        statArray: Array(6).fill(null),
        statModifiers: Array(6).fill(null),
        raceStatBonuses: [0, 0, 2, 0, 0, 0],
        classStatBonuses: Array(6).fill(0),
        finalStats: Array(6).fill(null),
        savingThrows: Array(6).fill(null),
        classStatSavingThrows: [0, 2],
        hitDice: '1d12',
        xp: 0,
        hp: 6,
        speed: 25,
        initiative: 0,
        proficiencyBonus: 2,
        languageids: [0, 1],
        armorClass: 10,
        proficiencies: [6, 14],
        toolProficiencies: [],
        equipment: [
          { id: 66, num: 1, },
          { id: 67, num: 1, },
          { id: 68, num: 1, },
          { id: 69, num: 1, },
          { id: 70, num: 1, },
          { id: 71, num: 1, },
          { id: 4, num: 4, },
          { id: 37, num: 1, },
        ],
        equipChoices: [
          { id: 0, chosen: false, selection: 0, remaining: 1, items: [] },
          { id: 1, chosen: false, selection: 0, remaining: 1, items: [] },
        ],
        currency: [0, 0, 0, 15, 0],
        traits: [0],
        ideals: [0],
        bonds: [0],
        flaws: [0],
        weaponModel: [],
      }
    }
  },
  computed: {
    allStatsAssigned: function () {
      return this.statRolls.length === 0;
    },
    allProficienciesChosen: function () {
      var numClassProfs = this.classProficiencies[this.output.classid].num;
      var numBackgroundProfs = this.backgroundProficiencies[this.output.backgroundid].profs.length;

      if (this.output.proficiencies.length < numClassProfs + numBackgroundProfs)
        return false;
      return true;
    },
    allLanguagesChosen: function () {
      var racelanguages = this.raceLanguages[this.races[this.output.raceid].raceid].languages.length;
      var extraLangs = this.raceExtraLanguages[this.output.raceid].extras;
      var numBackgroundLangs = this.getNumBackgroundLanguages();

      if (this.output.languageids.length < racelanguages + extraLangs + numBackgroundLangs)
        return false;
      return true;
    },
    allEquipmentChosen: function () {
      return this.equipmentChoiceModel.length === 0;
    },
    allChoicesFulfilled: function () {
      if (this.allEquipmentChosen && this.allLanguagesChosen && this.allProficienciesChosen && this.allStatsAssigned)
        return true;
      return false;
    },
    proficiencesLeftText: function () {
      // Nice text label of how many proficiencies left to select
      var numBackgroundProfs = this.backgroundProficiencies[this.output.backgroundid].profs.length;
      var numLeft = this.classProficiencies[this.output.classid].num - (this.output.proficiencies.length - numBackgroundProfs);

      return "Choose " + numLeft + " additional proficienc" + (numLeft > 1 ? "ies" : "y") + ":";
    },
    languagesLeftText: function () {
      // Nice text label to indicate how many additional languages you can choose
      var racelanguages = this.raceLanguages[this.races[this.output.raceid].raceid].languages.length;
      var extraLangs = this.raceExtraLanguages[this.output.raceid].extras;
      var numBackgroundLangs = this.getNumBackgroundLanguages();
      var numLeft = racelanguages + extraLangs + numBackgroundLangs - this.output.languageids.length;

      return "Choose " + numLeft + " additional language" + (numLeft > 1 ? "s" : "") + ":";
    },
    currentStatAssignmentText: function () {
      return this.statBlocks[this.statAssignmentIndex].text;
    },
    equipmentText: function () {
      // Nice text of selected equipment
      var i;
      var text = '';
      var equipIds = this.equipmentList;

      for (i = 0; i < equipIds.length; i++) {
        text += this.equipment[equipIds[i].id].text;
        if (equipIds[i].num > 1)
          text += ' (' + equipIds[i].num + ')';
        if (i < equipIds.length - 1)
          text += ', ';
      }

      return text;
    },
    equipmentTextList: function () {
      // Array of equipment text
      var i;
      var arr = [];
      var equipIds = this.equipmentList;

      for (i = 0; i < equipIds.length; i++) {
        var text = this.equipment[equipIds[i].id].text;
        if (equipIds[i].num > 1)
          text += ' (' + equipIds[i].num + ')';
        arr.push(text);
      }

      return arr;
    },
    equipmentList: function () {
      // Get list of equipment ID's and text
      var i;
      var equipIds = [];
      for (i = 0; i < this.output.equipment.length; i++) {
        var thisEquip = this.output.equipment[i];
        equipIds.push({ id: thisEquip.id, num: thisEquip.num });
      }

      for (i = 0; i < this.output.equipChoices.length; i++) {
        thisEquip = this.output.equipChoices[i];
        if (thisEquip.chosen === true) {
          var e;
          for (e = 0; e < thisEquip.items.length; e++) {
            var item = thisEquip.items[e];
            equipIds.push({ id: item.id, num: item.num });
          }
        }
      }

      return equipIds;
    },
    languagesText: function () {
      // Nice text of selected languages
      var i;
      var text = '';
      for (i = 0; i < this.output.languageids.length; i++) {
        var langName = this.languages[this.output.languageids[i]].text;
        text += langName;
        if (i < this.output.languageids.length - 1)
          text += ', ';
      }
      return text;
    },
    toolProficienciesText: function () {
      // Nice text of selected tool proficiencies
      var i;
      var text = '';
      for (i = 0; i < this.output.toolProficiencies.length; i++) {
        var name = this.toolProficiencies[this.output.toolProficiencies[i]].text;
        text += name;
        if (i < this.output.toolProficiencies.length - 1)
          text += ', ';
      }
      return text;
    },
    currencyText: function () {
      // Nice text of character currencies
      var text = '';
      if (this.output.currency[0] > 0)
        text += this.output.currency[0] + " CP ";
      if (this.output.currency[1] > 0)
        text += this.output.currency[1] + " SP ";
      if (this.output.currency[2] > 0)
        text += this.output.currency[2] + " EP ";
      if (this.output.currency[3] > 0)
        text += this.output.currency[3] + " GP ";
      if (this.output.currency[4] > 0)
        text += this.output.currency[4] + " PP ";
      return text;
    },
    traitText: function () {
      return this.personalityTraits[this.output.traits[0]].text;
    },
    idealText: function () {
      return this.ideals[this.output.ideals[0]].text;
    },
    bondText: function () {
      return this.bonds[this.output.bonds[0]].text;
    },
    flawText: function () {
      return this.flaws[this.output.flaws[0]].text;
    },
    level: function () {
      // Computed character level from XP
      if (this.output.xp === null || this.output.xp === '' || !this.isInt(this.output.xp))
        return 1;

      var i;
      var lev = 0;
      for (i = 0; i < this.xpLevels.length; i++) {
        if (this.output.xp >= this.xpLevels[i].xp)
          lev++;
        else
          break;
      }
      return lev;
    },
    hasBonusLanguages: function () {
      // Whether the current class/background combo is able to select additional languages
      var numBackgroundLangs = this.getNumBackgroundLanguages();
      return this.raceExtraLanguages[this.output.raceid].extras > 0 || numBackgroundLangs > 0;
    },
    availableLanguages: function () {
      // List of all available additional languages to choose from, excluding already selected
      var langs = [];
      var i;
      for (i = 0; i < this.languages.length; i++) {
        var lang = this.languages[i];
        if (!this.output.languageids.includes(lang.id))
          langs.push(lang);
      }
      return langs;
    },
    hasChosenAnyEquipment: function () {
      return this.equipmentChoiceModel.length < this.classEquipment[this.output.classid].equipChoices.length;
    },
    availableProficiencies: function () {
      // List of all available proficiencies to choose from, excluding already selected
      var profs = [];
      var i;
      var classProfs = this.classProficiencies[this.output.classid].profs;
      var backgroundProfs = this.backgroundProficiencies[this.output.backgroundid].profs;
      for (i = 0; i < this.skills.length; i++) {
        var prof = this.skills[i];

        if (!this.output.proficiencies.includes(i) && (classProfs.includes(i) || backgroundProfs.includes(i)))
          profs.push(prof);
      }
      return profs;
    },
    equipmentChoiceModel: function () {
      // List of all available equipment choices to choose from, excluding already selected
      var equipChoices = this.classEquipment[this.output.classid].equipChoices;
      var model = [];
      var i;
      for (i = 0; i < equipChoices.length; i++) {
        var equipChoice = equipChoices[i];
        var chosenChoices = this.output.equipChoices[i];
        if (chosenChoices.chosen === false || chosenChoices.chosen === true && chosenChoices.remaining > 0) {
          var modelRow = {
            id: equipChoice.id,
            choices: equipChoice.choices,
            chosen: chosenChoices.chosen,
            selection: chosenChoices.selection,
            remaining: chosenChoices.remaining
          };
          model.push(modelRow);
        }
      }
      return model;
    },
    generateButtonBindings: function () {
      if (!this.allChoicesFulfilled) {
        return {
          disabled: 'disabled'
        }
      }
      return {}
    }
  },
  methods: {
    isInt: function (x) {
      var y = parseInt(x, 10);
      return !isNaN(y);
    },
    d6: function () {
      // D6 roll
      return Math.floor(Math.random() * 6) + 1;
    },
    dX: function (x) {
      // DX roll
      return Math.floor(Math.random() * x) + 1;
    },
    statRoll: function () {
      // Standard stat roll: 4d6, subtract lowest, get total of remaining
      var diceArray = [...new Array(4)]
        .map(() => this.d6())
        .sort((a, b) => a - b)
        .reverse();
      diceArray.pop();
      return diceArray.reduce((a, b) => a + b, 0);
    },
    rollStats: function () {
      // Reset stats and create random stat array to assign from
      this.resetStats();
      this.statRolls = [...new Array(6)]
        .map(() => this.statRoll())
        .sort((a, b) => a - b)
        .reverse();
    },
    standardStats: function () {
      // Reset stats and use create standard stat array to assign from
      this.resetStats();
      this.statRolls = this.standardStatArray.slice();
    },
    allocateStat: function (val) {
      // Set the stat at index i to the value val
      this.output.statArray[this.statAssignmentIndex] = val;
      this.statRolls.splice(this.statRolls.indexOf(val), 1);
      this.statAssignmentIndex++;
      if (this.statAssignmentIndex === 5) {
        // Auto-assign last
        this.allocateStat(this.statRolls[0]);
        this.statAssignmentIndex = 0;
      }

      this.calculateStats();
    },
    setBackground: function (bg) {
      this.output.background = bg.text;
      this.output.backgroundid = bg.id;
      this.output.toolProficiencies = this.backgroundToolProficiencies[bg.id].profs.slice();
      this.output.currency = this.backgroundCurrency[this.output.backgroundid].currency.slice();

      this.resetEquipment();
      this.resetProficiencies();
      this.resetLanguages();
    },
    setClass: function (cls) {
      this.output.class = cls.text;
      this.output.classid = cls.id;
      this.output.classStatSavingThrows = this.savingThrows[cls.id].throws.slice();
      var hitDice = this.hitDice[cls.id];
      this.output.hitDice = hitDice.num + 'd' + hitDice.dice;

      this.resetEquipment();
      this.resetProficiencies();
      this.resetLanguages();
      this.standardStats();
    },
    setRace: function (race) {
      this.output.race = race.text;
      this.output.raceid = race.id;
      this.output.speed = this.speeds[race.id].value;
      this.output.raceStatBonuses = this.racialBonuses[race.id].bonuses;

      this.calculateStats();
      this.resetLanguages();
    },
    resetStats: function () {
      this.statAssignmentIndex = 0;
      this.output.statArray = Array(6).fill(null);
      this.output.statModifiers = Array(6).fill(null);
      this.output.finalStats = Array(6).fill(null);
    },
    resetLanguages: function () {
      this.output.languageids = this.raceLanguages[this.races[this.output.raceid].raceid].languages.slice();
    },
    resetProficiencies: function () {
      this.output.proficiencies = this.backgroundProficiencies[this.output.backgroundid].profs.slice();
    },
    resetEquipment: function () {
      var newEquip = [];
      var bgEquip = this.backgroundEquipment[this.output.backgroundid];
      var e;
      for (e = 0; e < bgEquip.equipids.length; e++) {
        var id = bgEquip.equipids[e];
        var eq = this.equipment[id];
        newEquip.push({ id: eq.id, num: 1 });
      }

      var classFixedEquip = this.classEquipment[this.output.classid].fixedEquip;
      if (typeof classFixedEquip !== 'undefined') {
        for (e = 0; e < classFixedEquip.length; e++) {
          var fixedEquip = classFixedEquip[e];
          eq = this.equipment[fixedEquip.id];
          newEquip.push({ id: eq.id, num: fixedEquip.num });
        }
      }

      this.output.equipment = newEquip;

      // Equipment choices
      var equipChoices = this.classEquipment[this.output.classid].equipChoices;
      var newEquipChoices = [];
      for (e = 0; e < equipChoices.length; e++) {
        var choice = equipChoices[e];
        newEquipChoices.push({ id: choice.id, chosen: false, selection: 0, remaining: 0, items: [] });
      }

      this.output.equipChoices = newEquipChoices;
    },
    addLanguage: function (id) {
      this.output.languageids.push(id);
    },
    addProficiency: function (id) {
      this.output.proficiencies.push(id);
    },
    selectEquipment: function (categoryId, choiceId, equipId) {
      var choiceCategory = this.output.equipChoices[categoryId];
      var equipModel = this.classEquipment[this.output.classid].equipChoices[categoryId];
      if (choiceCategory.chosen === false) {
        var equipChoice = equipModel.choices[choiceId];
        choiceCategory.chosen = true;
        choiceCategory.selection = choiceId;
        choiceCategory.remaining = equipChoice.num - 1;
        this.addChoiceCategoryEquip(choiceCategory, equipId);

        // If 2x or more of one item, automatically add remaining
        if (equipChoice.items.length === 1 && choiceCategory.remaining > 0) {
          var i;
          for (i = 0; i < choiceCategory.remaining; i++) {
            this.addChoiceCategoryEquip(choiceCategory, equipId);
          }
          choiceCategory.remaining = 0;
        }

        // Add extras
        if (typeof equipChoice.extras !== 'undefined') {
          var extras = equipChoice.extras;
          for (i = 0; i < extras.length; i++) {
            // Added to fixed equipment, might have complications
            this.output.equipment.push({ id: extras[i].id, num: extras[i].num });
          }
        }

      } else {
        choiceCategory.remaining--;
        choiceCategory.items.push(equipId);
      }
    },
    addChoiceCategoryEquip: function (equipChoice, equipId) {
      var itemChoice = equipChoice.items.find(item => item.id === equipId);
      if (typeof itemChoice !== 'undefined') {
        itemChoice.num++;
      } else {
        equipChoice.items.push({ id: equipId, num: 1 });
      }
    },
    getEquipmentName: function (id) {
      return this.equipment[id].text;
    },
    getNumBackgroundLanguages: function () {
      var backgroundLangs = this.backgroundLangauges.find(bgl => bgl.id === this.output.backgroundid);
      return backgroundLangs ? backgroundLangs.num : 0;
    },
    hasWeaponProficiency: function (equipment) {
      var classWeaps = this.classWeaponProficiencies[this.output.classid];
      if (classWeaps.categories.includes(equipment.type))
        return true;
      if (classWeaps.weapons.includes(equipment.id))
        return true;
      return false;
    },
    getModifier: function (val) {
      // Get stat modifier from lookup table
      var i;
      for (i = 0; i < this.statModifiers.length; i++) {
        var mod = this.statModifiers[i];
        if (val <= mod.val)
          return mod.modifier;
      }
      return this.statModifiers[this.statModifiers.length - 1].modifier;
    },
    formatModifier: function (val) {
      if (val > 0)
        return "+" + val;
      return val;
    },
    buildWeaponModel: function () {
      // Build model of weaponry chosen or included, with atk and damage type
      this.output.weaponModel = [];

      var i;
      var equipmentData;
      for (i = 0; i < this.output.equipChoices.length; i++) {
        var choiceItems = this.output.equipChoices[i].items;
        var x;
        for (x = 0; x < choiceItems.length; x++) {
          equipmentData = this.equipment[choiceItems[x].id];
          this.addWeaponModel(equipmentData);
        }
      }

      for (i = 0; i < this.output.equipment.length; i++) {
        equipmentData = this.equipment[this.output.equipment[i].id];
        this.addWeaponModel(equipmentData);
      }

      // Condense model (remove duplicates)
      var usedIds = [];
      var newWeaponModel = [];
      for (i = 0; i < this.output.weaponModel.length; i++) {
        var weapModel = this.output.weaponModel[i];
        if (!usedIds.includes(weapModel.id)) {
          usedIds.push(weapModel.id);
          newWeaponModel.push(weapModel);
        }
      }

      this.output.weaponModel = newWeaponModel;
    },
    addWeaponModel: function (equipmentData) {
      if (equipmentData.type === 0) { // Weapon
        var newWeap = {
          id: equipmentData.id,
          name: equipmentData.text,
          dice: equipmentData.dice,
          dmgType: this.damageTypes[equipmentData.damage].text,
          atkBonus: 0,
          dmgBonus: 0
        };

        // TODO: Calc attack bonus, this is going to be ugly
        var atkBonus = 0;
        var dmgBonus = 0;
        if (equipmentData.melee === true) { // TODO: Check finesse, use DEX if true
          atkBonus += this.output.statModifiers[0]; // STRmod
          dmgBonus += this.output.statModifiers[0];
        } else {
          atkBonus += this.output.statModifiers[1]; // DEXmod
          dmgBonus += this.output.statModifiers[1];
        }

        if (this.hasWeaponProficiency(equipmentData)) {
          atkBonus += this.output.proficiencyBonus;
        }

        newWeap.atkBonus = atkBonus;
        newWeap.dmgBonus = dmgBonus;

        this.output.weaponModel.push(newWeap);
      }
    },
    base64ToArrayBuffer: function (base64) {
      var binaryString = window.atob(base64);
      var binaryLen = binaryString.length;
      var bytes = new Uint8Array(binaryLen);
      for (var i = 0; i < binaryLen; i++) {
        var ascii = binaryString.charCodeAt(i);
        bytes[i] = ascii;
      }
      return bytes;
    },
    calculateStats: function () {
      // Add all stats together
      var i;
      for (i = 0; i < this.output.statArray.length; i++) {
        if (this.output.statArray[i] === null)
          continue;

        this.output.finalStats[i] = this.output.statArray[i] + this.output.raceStatBonuses[i];
        this.output.statModifiers[i] = this.getModifier(this.output.finalStats[i]);
        this.output.savingThrows[i] = this.output.statModifiers[i];
      }
    },
    generate: function () {

      // Initiative: DEX modifier
      this.output.initiative = this.output.statModifiers[1];

      // Base armor class: 10 + DEX modifier (does not include shield or armor)
      this.output.armorClass = 10 + this.output.statModifiers[1];

      // HP: Starting HP + CON
      this.output.hp = this.startingHp[this.output.classid].hp + this.output.statModifiers[2];
      this.output.level = this.level;

      this.output.proficiencyBonus = this.proficiencyLevels[this.output.level - 1].bonus;

      this.buildWeaponModel();

      // Add proficiency to saving throw stats
      var savingThrows = this.savingThrows[this.output.classid].throws.slice();
      var i;
      for (i = 0; i < 2; i++) {
        this.output.savingThrows[savingThrows[i]] += this.output.proficiencyBonus;
      }

      this.processPdf();
    },
    processPdf: function() {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', './DnD_5E_CharacterSheet - Form Fillable.pdf', true);
      xhr.responseType = 'arraybuffer';
      var process = this.fillPdfFields;
      xhr.onload = function() {
        if (this.status == 200) {
          process(this.response);
        } else {
          alert('Couldn\'t obtain the character sheet PDF blob.');
        }
      };
      xhr.send();
    },
    fillPdfFields: function(blob) {
      var fields = {};
      var o = this.output;
      fields['PlayerName'] = [o.playerName];
      fields['CharacterName'] = [o.characterName];
      fields['ClassLevel'] = [o.class + ' ' + this.level];
      fields['Race '] = [o.race];
      fields['Background'] = [o.background];

      var niceAlignment = o.alignmentLawChaos + ' ' + o.alignmentGoodEvil;
      if (niceAlignment === 'Neutral Neutral')
        niceAlignment = 'True Neutral';
      fields['Alignment'] = [niceAlignment];

      fields['XP'] = [o.xp];

      fields['STR'] = [this.formatModifier(o.statModifiers[0])];
      fields['DEX'] = [this.formatModifier(o.statModifiers[1])];
      fields['CON'] = [this.formatModifier(o.statModifiers[2])];
      fields['INT'] = [this.formatModifier(o.statModifiers[3])];
      fields['WIS'] = [this.formatModifier(o.statModifiers[4])];
      fields['CHA'] = [this.formatModifier(o.statModifiers[5])];

      fields['STRmod'] = [o.finalStats[0]];
      fields['DEXmod '] = [o.finalStats[1]];
      fields['CONmod'] = [o.finalStats[2]];
      fields['INTmod'] = [o.finalStats[3]];
      fields['WISmod'] = [o.finalStats[4]];
      fields['CHamod'] = [o.finalStats[5]];
      
      fields['HPMax'] = [o.hp];
      fields['Speed'] = [o.speed];
      fields['Initiative'] = [this.formatModifier(o.initiative)];

      fields['ST Strength'] = [this.formatModifier(o.savingThrows[0])];
      fields['ST Dexterity'] = [this.formatModifier(o.savingThrows[1])];
      fields['ST Constitution'] = [this.formatModifier(o.savingThrows[2])];
      fields['ST Intelligence'] = [this.formatModifier(o.savingThrows[3])];
      fields['ST Wisdom'] = [this.formatModifier(o.savingThrows[4])];
      fields['ST Charisma'] = [this.formatModifier(o.savingThrows[5])];

      var i;
      for (i = 0; i < 2; i++) {
        var savingThrowStat = o.classStatSavingThrows[i];
        switch(savingThrowStat) {
          case 0:
            fields['Check Box 11'] = [true];
            break;
          case 1:
            fields['Check Box 18'] = [true];
            break;
          case 2:
            fields['Check Box 19'] = [true];
            break;
          case 3:
            fields['Check Box 20'] = [true];
            break;
          case 4:
            fields['Check Box 21'] = [true];
            break;
          case 5:
            fields['Check Box 22'] = [true];
            break;
          default:
            break;
        }
      }

      fields['HDTotal'] = [o.hitDice];
      fields['ProfBonus'] = [o.proficiencyBonus];

      var profLangText = 'Languages: ' + this.languagesText;
      if (this.toolProficienciesText !== '')
        profLangText += '\n\nProficiencies: ' + this.toolProficienciesText;
      fields['ProficienciesLang'] = [profLangText];

      fields['AC'] = [o.armorClass];
      fields['Equipment'] = [this.equipmentText];

      //fields['CP'] = [o.currency[0]];
      //fields['SP'] = [o.currency[1]];
      //fields['EP'] = [o.currency[2]];
      fields['GP'] = [o.currency[3]];
      //fields['PP'] = [o.currency[4]];

      fields['PersonalityTraits '] = [this.traitText];
      fields['Ideals'] = [this.idealText];
      fields['Bonds'] = [this.bondText];
      fields['Flaws'] = [this.flawText];

      // https://rpg.stackexchange.com/questions/101169/how-does-passive-perception-work
      var pp = 10 + o.statModifiers[4];
      if (o.proficiencies.includes(11))
        pp += o.proficiencyBonus;

      fields['Passive'] = [pp];

      fields['Acrobatics'] = [this.formatModifier(o.statModifiers[1] + (o.proficiencies.includes(0) ? o.proficiencyBonus : 0))];
      fields['Animal'] = [this.formatModifier(o.statModifiers[4] + (o.proficiencies.includes(1) ? o.proficiencyBonus : 0))];
      fields['Arcana'] = [this.formatModifier(o.statModifiers[3] + (o.proficiencies.includes(2) ? o.proficiencyBonus : 0))];
      fields['Athletics'] = [this.formatModifier(o.statModifiers[0] + (o.proficiencies.includes(3) ? o.proficiencyBonus : 0))];
      fields['Deception '] = [this.formatModifier(o.statModifiers[5] + (o.proficiencies.includes(4) ? o.proficiencyBonus : 0))];
      fields['History '] = [this.formatModifier(o.statModifiers[3] + (o.proficiencies.includes(5) ? o.proficiencyBonus : 0))];
      fields['Insight'] = [this.formatModifier(o.statModifiers[4] + (o.proficiencies.includes(6) ? o.proficiencyBonus : 0))];
      fields['Intimidation'] = [this.formatModifier(o.statModifiers[5] + (o.proficiencies.includes(7) ? o.proficiencyBonus : 0))];
      fields['Investigation '] = [this.formatModifier(o.statModifiers[3] + (o.proficiencies.includes(8) ? o.proficiencyBonus : 0))];
      fields['Medicine'] = [this.formatModifier(o.statModifiers[4] + (o.proficiencies.includes(9) ? o.proficiencyBonus : 0))];
      fields['Nature'] = [this.formatModifier(o.statModifiers[3] + (o.proficiencies.includes(10) ? o.proficiencyBonus : 0))];
      fields['Perception '] = [this.formatModifier(o.statModifiers[4] + (o.proficiencies.includes(11) ? o.proficiencyBonus : 0))];
      fields['Performance'] = [this.formatModifier(o.statModifiers[5] + (o.proficiencies.includes(12) ? o.proficiencyBonus : 0))];
      fields['Persuasion'] = [this.formatModifier(o.statModifiers[5] + (o.proficiencies.includes(13) ? o.proficiencyBonus : 0))];
      fields['Religion'] = [this.formatModifier(o.statModifiers[3] + (o.proficiencies.includes(14) ? o.proficiencyBonus : 0))];
      fields['SleightofHand'] = [this.formatModifier(o.statModifiers[1] + (o.proficiencies.includes(15) ? o.proficiencyBonus : 0))];
      fields['Stealth '] = [this.formatModifier(o.statModifiers[1] + (o.proficiencies.includes(16) ? o.proficiencyBonus : 0))];
      fields['Survival'] = [this.formatModifier(o.statModifiers[4] + (o.proficiencies.includes(17) ? o.proficiencyBonus : 0))];

      // Proficiencies
      for (i = 0; i < o.proficiencies.length; i++) {
        var prof = o.proficiencies[i];
        switch(prof) {
          case 0:
            fields['Check Box 23'] = [true];
            break;
          case 1:
            fields['Check Box 24'] = [true];
            break;
          case 2:
            fields['Check Box 25'] = [true];
            break;
          case 3:
            fields['Check Box 26'] = [true];
            break;
          case 4:
            fields['Check Box 27'] = [true];
            break;
          case 5:
            fields['Check Box 28'] = [true];
            break;
          case 6:
            fields['Check Box 29'] = [true];
            break;
          case 7:
            fields['Check Box 30'] = [true];
            break;
          case 8:
            fields['Check Box 31'] = [true];
            break;
          case 9:
            fields['Check Box 32'] = [true];
            break;
          case 10:
            fields['Check Box 33'] = [true];
            break;
          case 11:
            fields['Check Box 34'] = [true];
            break;
          case 12:
            fields['Check Box 35'] = [true];
            break;
          case 13:
            fields['Check Box 36'] = [true];
            break;
          case 14:
            fields['Check Box 37'] = [true];
            break;
          case 15:
            fields['Check Box 38'] = [true];
            break;
          case 16:
            fields['Check Box 39'] = [true];
            break;
          case 17:
            fields['Check Box 40'] = [true];
            break;
          default:
            break;
        }
      }
      
      // Weapons
      for (i = 0; i < o.weaponModel.length; i++) {
        var weap = o.weaponModel[i];
        var weapAtkBonusStr = this.formatModifier(weap.atkBonus);
        var weapDmgStr = weap.dice + (weap.dmgBonus != 0 ? ' ' + this.formatModifier(weap.dmgBonus) : '') + ' ' + weap.dmgType;

        switch(i) {
          case 0:
            fields['Wpn Name'] = [weap.name];
            fields['Wpn1 AtkBonus'] = [weapAtkBonusStr];
            fields['Wpn1 Damage'] = [weapDmgStr];
            break;
          case 1:
              fields['Wpn Name 2'] = [weap.name];
              fields['Wpn2 AtkBonus '] = [weapAtkBonusStr];
              fields['Wpn2 Damage '] = [weapDmgStr];
            break;
          case 2:
              fields['Wpn Name 3'] = [weap.name];
              fields['Wpn3 AtkBonus  '] = [weapAtkBonusStr];
              fields['Wpn3 Damage '] = [weapDmgStr];
            break;
          default:
            break;
        }
      }

      // eslint-disable-next-line
      var filledPdf = pdfform().transform(blob, fields);
      var outBlob = new Blob([filledPdf], {type: 'application/pdf'});
      var fileName = 'DnD5e - Lvl' + this.level + ' ' + this.output.race + ' ' + this.output.class;
      if (this.output.playerName !== '')
        fileName += ' ' + this.output.playerName;

      saveAs(outBlob, fileName + '.pdf');
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
