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
        <a href="https://github.com/drogoganor/DnDCharacterSheetMaker" class="alert-link">GitHub page</a> for more details or to report issues.
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
          <div class="buttons has-addons">
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
          <div class="buttons has-addons">
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
          <div class="buttons has-addons">
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
          <div class="buttons has-addons">
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
          <div class="buttons has-addons">
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
      <div>
        <button type="button" class='button is-danger' v-on:click="standardStats">Standard Array</button>&nbsp;
        <button type="button" class='button is-danger' v-on:click="rollStats">Reroll</button>
      </div>
      <div class='tags are-medium'>
        <span
          v-for="roll in statRolls"
          v-bind:key="roll"
          class="tag is-info"
        >{{ roll }}</span>
      </div>
      <div class='columns'>
        <div class='column is-2'></div>
        <div class='column is-1'>Modifier</div>
        <div class='column is-1'>Total</div>
        <div class='column is-1'>Racial</div>
        <div class='column is-1'>Rolled</div>
        <div class='column'></div>
      </div>
      <div class='columns' id="stat-block" v-for="block in statBlocks" v-bind:key="block.id">
        <div class='column is-2'>{{ block.text }}</div>
        <div class='column is-1'>{{ formatModifier(output.statModifiers[block.id]) }}</div>
        <div class='column is-1'>{{ output.finalStats[block.id] }}</div>
        <div class='column is-1'>{{ output.raceStatBonuses[block.id] }}</div>
        <div class='column is-1'>{{ output.statArray[block.id] }}</div>
        <div class='column buttons are-small' v-if="output.statArray[block.id] == null">
          <button
            type="button"
            class="button is-success"
            v-for="r in statRolls"
            v-bind:key="r"
            v-on:click="allocateStat(block.id, r)"
          >{{ r }}</button>
        </div>
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
            <button type="button" class='button is-danger' v-on:click="resetProficiencies">Reset</button>
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
            <button type="button" class='button is-danger' v-on:click="resetEquipment">Reset</button>
          </div>
          <div v-for="choices in equipmentChoiceModel" v-bind:key="choices.id" class='buttons is-small'>
              <span class="row" v-for="choice in choices.choices" v-bind:key="choice.id">
                  <span class='tag'>{{ choice.num }}x</span>
                  <button type="button" v-for="item in choice.items" v-bind:key="item.id" class='button is-success' v-on:click="selectEquipment(choices.id, choice.id, item)">{{ getEquipmentName(item) }}</button>
                  <span class='tag' v-if="choice.id < choices.choices.length - 1">OR</span>
              </span>
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
            <button type="button" class='button is-danger' v-on:click="resetLanguages">Reset</button>
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

        >Generate PDF</button>
        <div v-if="allChoicesFulfilled"></div>
      </div>

      <div>&nbsp;</div>
    </div>
  </div>
</template>

<script src="./dnd.js"></script>

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
