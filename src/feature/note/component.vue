<template>
  <div class="skill-note">
    <btn @click="columns = []" class="reset">リセット</btn>

    <div class="stock">
      在庫
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
    </div>

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

        <div class="cell calculated-stock" v-for="i in rowNum">
          {{ calculatedStocks[i - 1] }}
        </div>
      </div>
      <div
        class="column"
        v-for="({ weapon, skills, bonuses }, i) of columns"
        :key="i"
      >
        <div class="cell weapon">
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
          class="cell bonus"
          v-for="bonus of bonuses"
        >
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
              {{ tdStock - 3 * j }}->{{ tdStock - 3 * (j + 1) }}
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
      .map((_, i) => stock.value.oricalcite - 20 * i)
      .map((v) => `${v + 20}->${v}`)
  } else if (props.mode === 'skill') {
    const s = { ...stock.value.tarredDevice }
    const css = []
    for (let i = 0; i < rowNum.value; i++) {
      const column = columns.value.find(({ skills }) => skills[i]?.selected)
      const focusType = column?.weapon.focusType ?? '会心'
      s[focusType] ??= s[focusType] ?? 0
      s[focusType] -= 3
      css.push({ [focusType]: `${s[focusType] + 3}->${s[focusType]}` })
    }
    return css.map((s) =>
      Object.entries(s)
        .map(([key, value]) => `${key[0]}:${value}`)
        .join(' ')
    )
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
</script>

<style scoped>
.skill-note {
  --cell-height: 2rem;
  --cell-width: v-bind('({ bonus: "20.8rem", skill: "22.5rem" }[mode])');
  > .table {
    display: flex;
    > .column {
      &.first {
        > .cell {
          width: 100%;
          padding: 0 1rem;
        }
      }
      > .cell {
        height: var(--cell-height);
        width: var(--cell-width);

        display: flex;
        justify-content: center;
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
          > input[type="radio"] {
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
          > .kind {
            flex-grow: 1;
          }
          > .element {
            width: 2rem;
          }
          > .focusType {
            width: 3.3rem;
          }
        }
        &.bonus {
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
