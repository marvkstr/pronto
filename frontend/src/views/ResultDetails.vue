<template lang="pug">
  .results
    .ui.grid.container
      .row
        .two.wide.tablet.two.wide.computer.sixteen.wide.mobile.right.column
        .twelve.wide.tablet.twelve.wide.computer.sixteen.wide.mobile.centered.column
          h1 {{ result.label }}
          p {{ result.name }}
            i.icon.copy.outline.link(
              @click.stop.prevent="copyToClipboard(result.name)",
              title="Copy"
            )
            a(
              :href="result.name", 
              target="_blank"
            )
              i.icon.external.alternate.link(
                title="Open definition in own ontology"
              )
          br 
          table.ui.very.basic.celled.table
            thead
              tr
                th Predicate
                th Object
            tbody
              template(v-for="[k, vArr] in Object.entries(groupedResults)")
                tr
                  td.preds(:rowspan="vArr.length")
                    h4.ui.header
                      .content
                        | {{ prefix(k) }}
                        .sub.header
                          | {{ k }}
                  td
                    a(
                      v-html="prettyProp(vArr[0])",
                      :title="vArr[0]",
                      :href="vArr[0]",
                      target="_blank"
                    )

                  
                tr(v-for="(obj, idx) in vArr", v-if="idx > 0")
                  td.result-cell
                    a(
                      v-html="prettyProp(obj)", 
                      :title="obj",
                      :href="obj",
                      target="_blank"
                    )
                  
        .two.wide.tablet.two.wide.computer.sixteen.wide.mobile.right.column
          .ui.animated.button(
            tabindex='0'
            @click="$router.go(-1)"
          )
            .visible.content Go back
            .hidden.content
              i.left.arrow.icon


</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { prefixes, copyToClipboard } from '@/utils';
import { OntologyResult } from '@/types';

@Component({
  beforeRouteEnter(to, from, next) {
    next(vm => {
      // @ts-ignore
      if (!vm.result) {
        vm.$router.go(-1);
      }
    });
  },
})
export default class ResultDetails extends Vue {
  @Prop({ required: true }) result!: OntologyResult;

  // swap uri with prefixes
  prefixes = Object.entries(prefixes).reduce((acc, entry) => {
    const [key, value] = entry;
    acc[value] = key;
    return acc;
  }, {} as { [root: string]: string });

  get groupedResults() {
    return this.result.rest.reduce((prev, curr) => {
      if (!prev[curr.predicate.id]) prev[curr.predicate.id] = [];
      prev[curr.predicate.id] = [...prev[curr.predicate.id], curr.object.id];
      return prev;
    }, {} as { [key: string]: Array<string> });
  }

  copyToClipboard = copyToClipboard;
  prefix(s: string) {
    try {
      const re = s.match(/(\/|#)(?:.(?!\/|#))+$/);
      if (!re) return s;
      const [root, prop] = [s.slice(0, re['index']! + 1), s.slice(re['index']! + 1)];
      if (!this.prefixes[root]) return s;
      return `${this.prefixes[root]}:${prop}`;
    } catch {
      return s;
    }
  }
  prettyProp(prop: string) {
    if (prop.indexOf('@') > 0) {
      const s = prop.split('@');
      // TODO: DB only contains data in English. Include `<span>lang:${s[1]}</span>` when adding more languages
      return s[0].slice(1, -1);
    }
    return this.prefix(prop);
  }
}
</script>

<style lang="scss" scoped>
a > i {
  color: #2c3e50 !important;
}
.preds {
  vertical-align: top;
}
table.ui {
  margin-bottom: 100px !important;
}
.result-cell {
  padding-left: 0.78571429em !important;
  border-left: 1px solid rgba(34, 36, 38, 0.1) !important;
}
@media only screen and (max-width: 767px) {
  .ui.table:not(.unstackable) td:first-child {
    font-weight: 400;
  }
}
</style>