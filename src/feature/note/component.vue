<template>
  <div class="note">
    <div class="buttons">
      <btn
        @click="
          () => {
            const calculatedStock = calculatedStocks[processedNum - 1]?.value
            if (!calculatedStock) {
              return
            }
            // @ts-expect-error
            stock[
              (
                {
                  bonus: 'oricalcite',
                  skill: 'tarredDevice',
                } as const
              )[mode]
            ] = calculatedStock
            columns.forEach((column) => {
              column.bonuses.splice(0, processedNum)
              column.skills.splice(0, processedNum)
            })
            processedNum = 0
          }
        "
        class="confirm"
      >
        在庫確定
      </btn>
      <btn @click="columns = []" class="reset">リセット</btn>
    </div>

    <fieldset class="stock">
      <legend>
        {{
          // @ts-expect-error
          { skill: '油濁した遺装置' }[mode] ?? ''
        }}在庫
      </legend>
      <div v-if="mode === 'bonus'" class="oricalcite">
        <label>
          ナナイロカネ:
          <text-field
            :model-value="String(stock.oricalcite)"
            @update:model-value="
              (v) => {
                const num = Number(v)
                if (Number.isNaN(num)) {
                  return
                }
                stock.oricalcite = num
              }
            "
          />
        </label>
      </div>
      <div v-if="mode === 'skill'" class="tarred-device">
        <label v-for="focusType of weaponDef.focusTypes">
          {{ focusType }}:
          <text-field
            :model-value="String(stock.tarredDevice[focusType] ?? 0)"
            @update:model-value="
              (v) => {
                const num = Number(v)
                if (Number.isNaN(num)) {
                  return
                }
                stock.tarredDevice[focusType] = num
              }
            "
          />
        </label>
      </div>
    </fieldset>

    <details class="favorite-skills" v-if="mode === 'skill'">
      <summary>お気に入りスキル設定</summary>
      <div class="settings">
        <div
          v-for="{ fsKey, wdKey } of [
            { fsKey: 'series', wdKey: 'seriesSkills' },
            { fsKey: 'group', wdKey: 'groupSkills' },
          ] as const"
          :key="fsKey"
          :class="fsKey"
        >
          <label v-for="skill of weaponDef[wdKey]">
            <input
              type="checkbox"
              :checked="
                favoriteSkillSets[fsKey].has(
                  // @ts-expect-error
                  skill
                )
              "
              @change="
                (e) => {
                  const {
                    // @ts-expect-error
                    checked,
                  } = e.target ?? {}
                  const exists = favoriteSkillSets[fsKey].has(
                    // @ts-expect-error
                    skill
                  )
                  if (checked && !exists) {
                    favoriteSkills[fsKey].push(
                      // @ts-expect-error
                      skill
                    )
                  } else if (!checked && exists) {
                    // @ts-expect-error
                    favoriteSkills[fsKey] = favoriteSkills[fsKey].filter(
                      (s) => s !== skill
                    )
                  }
                }
              "
            />
            {{ skill }}
          </label>
        </div>
      </div>
    </details>

    <div class="table">
      <div class="column first">
        <btn
          class="cell"
          @click="
            columns.push({
              weapon: { ...(columns.at(-1)?.weapon ?? {}) },
              bonuses: Array(
                Math.max(1, ...columns.map(({ bonuses }) => bonuses.length))
              )
                .fill(null)
                .map(() => ({
                  values: [
                    undefined,
                    undefined,
                    undefined,
                    undefined,
                    undefined,
                  ],
                })),
              skills: Array(
                Math.max(1, ...columns.map(({ skills }) => skills.length))
              )
                .fill(null)
                .map(() => ({})),
            })
          "
        >
          +
        </btn>

        <div
          v-for="i in rowNum"
          :class="{
            'cell': true,
            'selected-row': columns.some(
              ({ bonuses, skills }) =>
                bonuses[i - 1]?.selected || skills[i - 1]?.selected
            ),
          }"
        >
          <label>
            <input
              type="checkbox"
              :checked="i <= processedNum"
              @change="
                (e) => {
                  processedNum =
                    i +
                    // @ts-expect-error
                    (e.target?.checked ? 0 : -1)
                }
              "
            />
            <span>{{ calculatedStocks[i - 1]?.label }}</span>
          </label>
        </div>
      </div>
      <div
        v-for="({ weapon, skills, bonuses }, i) of columns"
        :class="{
          'column': true,
          'selected-column':
            bonuses.some(({ selected }) => selected) ||
            skills.some(({ selected }) => selected),
        }"
        :key="i"
      >
        <div class="cell weapon">
          <btn @click="() => columns.splice(i, 1)" class="delete">x</btn>
          <slc v-model="weapon.kind" class="kind" :items="weaponDef.kinds" />
          <slc
            v-model="weapon.element"
            class="element"
            :items="weaponDef.elements"
          />
          <slc
            v-model="weapon.focusType"
            class="focusType"
            :items="weaponDef.focusTypes"
          />
        </div>
        <div
          v-if="mode === 'bonus'"
          :class="{
            cell: true,
            bonus: true,
            selected: bonus.selected,
          }"
          v-for="(bonus, j) of bonuses"
        >
          <label>
            <input
              type="radio"
              :checked="bonus.selected"
              @change="
                () =>
                  columns.forEach((column, k) => {
                    const bonus = column.bonuses[j]
                    if (!bonus) {
                      return
                    }
                    bonus.selected = k === i
                  })
              "
            />
          </label>
          <slc
            v-for="(_, i) in bonus.values"
            v-model="bonus.values[i]"
            :class="{
              series: true,
              lose: bonus.values[i] === 'はずれ',
            }"
            :items="weaponDef.bonuses"
            @click="fillBonus(bonus)"
            @focus="fillBonus(bonus)"
          />
        </div>
        <div
          v-else-if="mode === 'skill'"
          :class="{
            cell: true,
            skill: true,
            selected: skill.selected,
          }"
          v-for="(skill, j) of skills"
        >
          <label>
            <input
              type="radio"
              :checked="skill.selected"
              @change="
                () =>
                  columns.forEach((column, k) => {
                    const skill = column.skills[j]
                    if (!skill) {
                      return
                    }
                    skill.selected = k === i
                  })
              "
            />
            <div
              v-for="tdStock of [
                weapon.focusType
                  ? stock.tarredDevice[weapon.focusType]
                  : undefined,
              ].filter((v) => v != null)"
            >
              ->{{ tdStock - 3 * (j + 1) }}
            </div>
          </label>
          <slc
            v-model="skill.series"
            :class="{
              series: true,
              lose: skill.series === 'はずれ',
            }"
            @click="fillSkill(skill)"
            @focus="fillSkill(skill)"
            :items="weaponDef.seriesSkills"
            :fav-item-set="favoriteSkillSets.series"
          />
          <slc
            v-model="skill.group"
            :class="{
              group: true,
              lose: skill.group === 'はずれ',
            }"
            @click="fillSkill(skill)"
            @focus="fillSkill(skill)"
            :items="weaponDef.groupSkills"
            :fav-item-set="favoriteSkillSets.group"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, watch } from 'vue'

import TextField from '../../component/text-field/component.vue'
import Slc from '../../component/select/component.vue'
import Btn from '../../component/button/component.vue'

import { type Weapon, useWeaponDef } from '../../composable/weapon-defs-usable'

const weaponDef = useWeaponDef()

const props = defineProps<{
  mode: 'bonus' | 'skill'
}>()

const columns = defineModel<Column[]>('columns', {
  default: [],
})

const stock = defineModel<Stock>('stock', {
  default: {
    oricalcite: 0,
    tarredDevice: {},
  },
})
const processedNum = defineModel<number>('processedNum', {
  default: 0,
})

const favoriteSkills = defineModel<FavoriteSkills>('favoriteSkills', {
  default: { series: [], group: [] },
})
const favoriteSkillSets = computed(() => ({
  series: new Set(favoriteSkills.value.series),
  group: new Set(favoriteSkills.value.group),
}))

const rowNum = computed(
  () =>
    columns.value[0]?.[
      (
        {
          bonus: 'bonuses',
          skill: 'skills',
        } as const
      )[props.mode]
    ].length ?? 0
)

const calculatedStocks = computed(() => {
  if (props.mode === 'bonus') {
    return Array(rowNum.value)
      .fill(null)
      .map((_, i) => stock.value.oricalcite - 20 * (i + 1))
      .map((v) => ({ value: v, label: `->${v}` }))
  } else if (props.mode === 'skill') {
    const s = { ...stock.value.tarredDevice }
    const css = []
    for (let i = 0; i < rowNum.value; i++) {
      const column = columns.value.find(({ skills }) => skills[i]?.selected)
      const focusType =
        column?.weapon.focusType ??
        (['会心', '属性', '攻撃'] as const).find(
          (type) => 3 <= (s[type] ?? 0)
        ) ??
        '会心'
      s[focusType] ??= s[focusType] ?? 0
      s[focusType] -= 3
      css.push({
        value: { ...s },
        label: `${focusType}:->${s[focusType]}`,
      })
    }
    return css
  }
  return []
})

watch(
  () =>
    columns.value.some((column) =>
      column.bonuses[column.bonuses.length - 1]?.values.some((v) => Boolean(v))
    ),
  (nv) => {
    if (!nv) {
      return
    }
    columns.value.forEach((column) =>
      column.bonuses.push({
        values: [undefined, undefined, undefined, undefined, undefined],
      })
    )
  }
)

watch(
  () =>
    columns.value.some((column) => {
      const { series, group } = column.skills[column.skills.length - 1] ?? {}
      return series || group
    }),
  (nv) => {
    if (!nv) {
      return
    }
    columns.value.forEach((column) => column.skills.push({}))
  }
)

function fillBonus(
  bonus: (typeof columns)['value'][number]['bonuses'][number]
) {
  bonus.values.forEach((_, i) => {
    bonus.values[i] ??= 'はずれ'
  })
}

function fillSkill(skill: (typeof columns)['value'][number]['skills'][number]) {
  skill.group ??= 'はずれ'
  skill.series ??= 'はずれ'
}
</script>

<script lang="ts">
export type Column = {
  weapon: {
    kind?: Weapon['kinds']
    element?: Weapon['elements']
    focusType?: Weapon['focusTypes']
  }
  bonuses: {
    selected?: boolean
    values: (Weapon['bonuses'] | undefined)[]
  }[]
  skills: {
    selected?: boolean
    series?: Weapon['seriesSkills']
    group?: Weapon['groupSkills']
  }[]
}

export type Stock = {
  tarredDevice: { [focusType in Weapon['focusTypes']]?: number }
  oricalcite: number
}
export type FavoriteSkills = {
  series: Weapon['seriesSkills'][]
  group: Weapon['groupSkills'][]
}
</script>

<style scoped>
.note {
  display: flex;
  flex-direction: column;
  gap: 8px;

  > * {
    width: fit-content;
  }

  > .buttons {
    display: flex;
    gap: 8px;
    align-items: center;
    > button {
      height: 2.3rem;
      padding: 0 1rem;
    }
  }
  > fieldset.stock {
    border: solid 1px var(--c-border);
  }
  > details.favorite-skills {
    > .settings {
      display: flex;
      > div {
        display: flex;
        flex-direction: column;
      }
    }
  }

  --cell-height: 2rem;
  --cell-width: v-bind('({ bonus: "22.7rem", skill: "20.5rem" }[mode])');
  > .table {
    display: flex;
    > .column {
      &.selected-column {
        > .cell.weapon {
          background-color: var(--c-active-bg);
        }
      }
      &.first {
        position: sticky;
        left: 0;
        > .cell {
          min-width: max-content;
          width: 100%;
          padding: 0 1rem;
          &.selected-row {
            background-color: var(--c-active-bg);
          }
        }
      }
      > .cell {
        background-color: white;

        height: var(--cell-height);
        width: var(--cell-width);

        display: flex;
        justify-content: flex-start;
        align-items: center;

        border: solid 1px var(--c-border);
        &:is(button) {
          width: 3rem;
        }

        label {
          height: 100%;
          display: flex;
          justify-content: center;
          align-items: center;
          > input[type='radio'] {
            margin: 0 4px;
          }
        }
        select {
          border: none;
          height: 100%;
          &.lose {
            color: lightgray;
          }
        }
        &.weapon {
          > .delete {
            height: 100%;
            width: 2rem;
            border: none;
          }
          > .kind {
            flex-grow: 1;
          }
          > .element {
            width: 3rem;
          }
          > .focusType {
            width: 3.3rem;
          }
        }
        &.bonus {
          > label {
            width: 3rem;
          }
          > * {
            width: 20%;
          }
        }
        &.skill {
          &.selected {
            background-color: var(--c-active-bg);
          }
          .series {
            flex-grow: 1;
          }
          .group {
            min-width: 7rem;
          }
        }
      }
    }
  }
}
</style>
