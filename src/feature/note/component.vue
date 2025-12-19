<template>
  <div class="skill-note">
    <btn @click="columns = []" class="reset">リセット</btn>

    <div class="table">
      <div class="column">
        <btn
          class="cell"
          @click="
            columns.push({
              weapon: { ...(columns.at(-1)?.weapon ?? {}) },
              bonuses: Array(
                Math.max(1, ...columns.map(({ bonuses }) => bonuses.length))
              )
                .fill(null)
                .map(() => [
                  undefined,
                  undefined,
                  undefined,
                  undefined,
                  undefined,
                ]),
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
        <div v-if="mode === 'skill'" class="cell skill" v-for="skill of skills">
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
            :items="weaponDef.groupSkills"
          />
        </div>
        <div
          v-else-if="mode === 'bonus'"
          class="cell bonus"
          v-for="bonus of bonuses"
        >
          <slc
            v-for="(_, i) in bonus"
            v-model="bonus[i]"
            :class="{
              series: true,
              lose: bonus[i] === 'はずれ',
            }"
            :items="weaponDef.bonuses"
            @click="fillBonus(bonus)"
            @focus="fillBonus(bonus)"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { watch } from 'vue'

import Slc from '../../component/select/component.vue'
import Btn from '../../component/button/component.vue'

import { type Weapon, useWeaponDef } from '../../composable/weapon-defs-usable'

const weaponDef = useWeaponDef()

defineProps<{
  mode: 'skill' | 'bonus'
}>()

const columns = defineModel<Column[]>('columns', {
  default: [],
})

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

watch(
  () =>
    columns.value.some((column) =>
      column.bonuses[column.bonuses.length - 1]?.some((v) => Boolean(v))
    ),
  (nv) => {
    if (!nv) {
      return
    }
    columns.value.forEach((column) =>
      column.bonuses.push([
        undefined,
        undefined,
        undefined,
        undefined,
        undefined,
      ])
    )
  }
)

function fillSkill(skill: (typeof columns)['value'][number]['skills'][number]) {
  skill.group ??= 'はずれ'
  skill.series ??= 'はずれ'
}

function fillBonus(
  bonus: (typeof columns)['value'][number]['bonuses'][number]
) {
  bonus.forEach((_, i) => {
    bonus[i] ??= 'はずれ'
  })
}
</script>

<script lang="ts">
export type Column = {
  weapon: {
    kind?: Weapon['kinds']
    element?: Weapon['elements']
    focusType?: Weapon['focusTypes']
  }
  skills: {
    series?: Weapon['seriesSkills']
    group?: Weapon['groupSkills']
  }[]
  bonuses: ((Weapon['bonuses'] | undefined)[] & { length: 5 })[]
}
</script>

<style scoped>
.skill-note {
  --cell-height: 2rem;
  --cell-width: v-bind('({ bonus: "21rem", skill: "16.3rem" }[mode])');
  > .table {
    display: flex;
    > .column {
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
        &.skill {
          .series {
            flex-grow: 1;
          }
          .group {
            min-width: 7rem;
          }
        }
        &.bonus {
          > * {
            width: 20%;
          }
        }
      }
    }
  }
}
</style>
