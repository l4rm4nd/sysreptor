<template>
  <div class="pa-4">
    <v-expansion-panels v-if="group" multiple tile hover>
      <v-expansion-panel v-if="messageGroups.length === 0" class="error-group" readonly>
        <v-expansion-panel-header hide-actions>
          <div class="error-header text-body-1">
            <v-icon color="green" class="mr-2">mdi-checkbox-marked</v-icon>
            No errors or warnings detected.
          </div>
        </v-expansion-panel-header>
      </v-expansion-panel>

      <v-expansion-panel v-for="msgGroup, groupIdx in messageGroups" :key="'group-' + groupIdx" class="error-group">
        <v-expansion-panel-header>
          <div class="error-header text-body-1">
            <v-chip class="mr-2" :class="'level-' + msgGroup.level" small label>{{ msgGroup.level.toUpperCase() }}</v-chip>
            {{ msgGroup.message }}
            <v-spacer />
            <span class="mr-2">&times;{{ msgGroup.entries.length }}</span>
          </div>
        </v-expansion-panel-header>

        <v-expansion-panel-content class="pl-8">
          <div v-for="msg, entryIdx in msgGroup.entries" :key="'group-' + groupIdx + '-' + entryIdx" class="mb-2">
            <!-- {{ msg.message }} -->
            <span v-if="msg.location" class="error-location">
              <slot name="location" :msg="msg">
                in {{ msg.location.type }} 
                <template v-if="msg.location.name">{{ msg.location.name }}"</template>
                <template v-if="msg.location.path">{{ msg.location.path }}</template>
              </slot>
            </span>

            <pre v-if="msg.details" class="error-details ml-4">{{ msg.details }}</pre>
            <v-divider />
          </div>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>

    <div v-else v-for="msg, idx in messageList" :key="'list' + idx" class="mb-4">
      <div class="error-header">
        <v-chip class="ma-2 mt-0" :color="{'error': 'red', 'warning': 'yellow'}[msg.level]" small label>{{ msg.level.toUpperCase() }}</v-chip>
        <p class="error-message">
          <slot name="message" :msg="msg">
            {{ msg.message }}
            <span v-if="msg.location && msg.location.name" class="error-location">
              in "{{ msg.location.name }}"
              <template v-if="msg.location.path">{{ msg.location.path }}</template>
            </span>
          </slot>
        </p>
      </div>

      <pre v-if="msg.details" class="error-details ml-4">{{ msg.details }}</pre>
    </div>
  </div>
</template>

<script>
import { groupBy, sortBy } from 'lodash';

export default {
  props: {
    value: {
      type: [Object, Array],
      required: true,
    },
    group: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    messageList() {
      let out = [];
      if (Array.isArray(this.value)) {
        out = this.value.flatMap(i => (i?.error || []).concat(i?.warning || []));
      } else {
        out = (this.value?.error || []).concat(this.value?.warning || []);
      }
      return sortBy(out, [m => this.levelToNumber(m.level), 'message']);
    },
    messageGroups() {
      // Group by level and message text
      const grouped = groupBy(this.messageList, m => [m.level, m.message]);
      const formattedGroups = Object.values(grouped)
        .map(msgs => ({
          level: msgs[0].level,
          message: msgs[0].message,
          entries: msgs
        }));
      const sorted = sortBy(formattedGroups, [g => this.levelToNumber(g.level), 'message']);
      return sorted;
    },
  },
  methods: {
    levelToNumber(level) {
      const out = ['error', 'warning', 'info'].indexOf(level);
      if (out < 0) {
        return 9;
      }
      return out;
    }
  }
}
</script>

<style lang="scss" scoped>
.error-header {
  margin-bottom: 0;
  display: flex;
}
.error-message {
  display: inline-block;
  flex-grow: 1;
  margin-bottom: 0;
}
.error-location {
  font-size: smaller;
}
.error-details {
  font-size: small;
  white-space: pre-wrap;
}

.error-group {
  background-color: inherit !important;
}

.level-error {
  background-color: map-get($red, 'base') !important;
  color: white !important;
}
.level-warning {
  background-color: map-get($yellow,  'base') !important;
  color: black !important;
}
</style>
