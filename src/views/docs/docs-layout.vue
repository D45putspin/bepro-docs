<template>
  <div class="d-sm-flex">
    <div class="" ref="membersList">
      <doc-item class="pt-1" v-if="owner" :item="owner" />
      <template v-for="[kind, docs] in Object.entries(membersOf || {})">
        <ul :key="kind">
          <li><doc-item-kind :kind="kind" /></li>
          <ul>
            <li v-for="doc in docs" :key="doc.name" :id="doc.name">
              <doc-item :item="doc" />
            </li>
          </ul>
        </ul>
      </template>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Vue, Watch } from "vue-property-decorator";
import MembersList from "@components/members-list.vue";
import DocItemKind from "@components/doc-item-kind.vue";
import DocItem from "@components/doc-item.vue";
import { docsFind } from "@helpers/docs-find";
import WatchDocNameParam from "@components/mixins/watch-doc-name-param";
import { JsonDoc } from "@objects/faces/jsdocjson";

@Component({
  components: { DocItem, DocItemKind, MembersList },
})
export default class DocsLayout extends Mixins(WatchDocNameParam) {
  owner: JsonDoc | null = null;

scrollToParamsName() {
  console.log("ehehheheheh")
    return (this.$refs.membersList as Element).querySelector(`#${this.$route.params.name}`)?.scrollIntoView()
  }
  navigateTo(memberName?: string) {
    if (!memberName) return this.scrollToParamsName;

    if (
      !docsFind(null, [...Object.values(this.membersOf!)].flat(), {
        name: memberName,
      }).length
    )
      this.$router.push({
        name: this.$route.name!,
        params: {
          ...(this.$route.params?.name
            ? { name: this.$route.params?.name }
            : {}),
        },
      });
    else
      (this.$refs.membersList as Element)
        .querySelector(`li#${memberName}`)
        ?.scrollIntoView();
  }

  @Watch(`$route.params.member`) watchMemberName(
    newValue: string,
    oldValue: string
  ) {
    if (newValue === oldValue) return;

    this.navigateTo(newValue);
  }

  mounted() {
    this.setMemberOfObjectTo(this.$route.params.name);
    if (this.$route.params?.member) this.navigateTo(this.$route.params.member);
  }
}
</script>

