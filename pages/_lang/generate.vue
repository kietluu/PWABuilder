<template>
<section>
  <GeneratorMenu />

  <div v-if="manifest$">
    <div class="l-generator-step">
      <div class="l-generator-semipadded">
        <div class="l-generator-form pure-u-1 pure-u-md-1-2">
          <h4 class="l-generator-subtitle">
            {{ $t("generate.subtitle") }}
          </h4>
          <h4 class="l-generator-subtitle l-generator-subtitle--last">
            {{ $t("generate.instructions") }}
          </h4>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.name") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#name-member" target="_blank">[?]</a>
            </label>
            <input class="l-generator-input" v-model="manifest$.name" @change="onChangeSimpleInput()" type="text">
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.short_name") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#short_name-member" target="_blank">[?]</a>
            </label>
            <input class="l-generator-input" v-model="manifest$.short_name" @change="onChangeSimpleInput()" name="short_name" type="text">
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.description") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#description-member" target="_blank">[?]</a>
            </label>
            <input class="l-generator-input" v-model="manifest$.description" @change="onChangeSimpleInput()" name="description" type="text">
          </div>

          <Modal :title="$t('generate.upload_title')" ref="iconsModal" @submit="onSubmitIconModal" @cancel="onCancelIconModal">
            <div class="l-generator-box">
              <span class="l-generator-label">{{ $t("generate.upload_image") }}</span>
              <label class="l-generator-input l-generator-input--fake is-disabled" for="modal-file">
                {{ iconFile && iconFile.name ? iconFile.name : $t("generate.choose_file") }}
              </label>
              <input id="modal-file" @change="onFileIconChange" class="l-hidden" type="file">
            </div>

            <div class="l-generator-field">
              <label>
                <input type="checkbox" v-model="iconCheckMissing"> {{ $t("generate.generate_missing") }}
              </label>
            </div>
          </Modal>
          <div class="l-generator-field logo-upload">
            <label class="l-generator-label">{{ $t("generate.icon_url") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#icons-member" target="_blank">[?]</a>
            </label>
            <div>
              <input class="l-generator-input" placeholder="http://example.com/image.png or /images/example.png" type="url" v-model="newIconSrc">

              <div class="button-holder icons">
                <div class="l-inline">
                  <button class="pwa-button pwa-button--text" @click="onClickUploadIcon()">
                    {{ $t("generate.upload") }}
                  </button>
                </div>
                <button class="pwa-button pwa-button--text pwa-button--right" @click="onClickAddIcon()">
                  {{ $t("generate.add_icon") }}
                </button>
              </div>

              <p class="l-generator-error" v-if="error">
                <span class="icon-exclamation"></span>
                {{ $t(error) }}
              </p>

              <div class="pure-g l-generator-table">
                <div class="pure-u-10-24 l-generator-tableh">{{ $t("generate.preview") }}</div>
                <div class="pure-u-8-24 l-generator-tableh">{{ $t("generate.size") }}</div>
                <div class="pure-u-1-8"></div>
                <div class="pure-u-1-8"></div>

                <div class="pure-u-1" v-for="icon in icons" :key="icon.src">
                  <div class="pure-u-10-24 l-generator-tablec">
                    <a target="_blank" :href="icon.src">
                      <img class="icon-preview" :src="icon.src">
                    </a>
                  </div>
                  <div class="pure-u-8-24 l-generator-tablec">
                    {{icon.sizes}}
                  </div>
                  <div class="pure-u-1-8 l-generator-tablec" :title="$t('generate.icon_autogenerated')">
                    <span class="icon-magic" ng-if="icon.generated"></span>
                  </div>
                  <div class="pure-u-1-8 l-generator-tablec l-generator-tablec--right" @click="onClickRemoveIcon(icon)">
                    <span class="l-generator-close" :title="$t('generate.remove_icon')">
                      <i aria-hidden="true">
                        <span class="icon-times"></span>
                      </i>
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.scope") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#scope-member" target="_blank">[?]</a>
            </label>
            <input class="l-generator-input" v-model="manifest$.scope" @change="onChangeSimpleInput()" type="text">
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.start_url") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#start_url-member" target="_blank">[?]</a>
            </label>
            <input class="l-generator-input" v-model="manifest$.start_url" @change="onChangeSimpleInput()" type="text">
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.display") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#display-member" target="_blank">[?]</a>
            </label>
            <select class="l-generator-input l-generator-input--select" v-model="manifest$.display" @change="onChangeSimpleInput()">
              <option v-for="display in displaysNames" :value="display" :key="display">{{display}}</option>
            </select>
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.orientation") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#orientation-member" target="_blank">[?]</a>
            </label>
            <select class="l-generator-input l-generator-input--select" v-model="manifest$.orientation" @change="onChangeSimpleInput()">
              <option v-for="orientation in orientationsNames" :value="orientation" :key="orientation">{{orientation}}</option>
            </select>
          </div>
          <div class="l-generator-field">
            <label class="l-generator-label">{{ $t("generate.language") }}
              <a class="l-generator-link" href="https://www.w3.org/TR/appmanifest/#lang-member" target="_blank">[?]</a>
            </label>
            <select class="l-generator-input l-generator-input--select" v-model="manifest$.lang">
              <option v-for="language in languagesNames" :value="language" :key="language" @change="onChangeSimpleInput()">{{language}}</option>
            </select>
          </div>
          <div>
            <ColorSelector />
          </div>

          <div>
            <input type="checkbox" id="related-applications-field" class="l-generator-togglecheck is-hidden">
            <label class="l-generator-toggle" for="related-applications-field">
              <h4 class="l-generator-subtitle l-generator-subtitle--toggleable">{{ $t("generate.specify_application") }}</h4>
            </label>
            <div class="l-generator-field l-generator-field--toggle">
              <RelatedApplications />
            </div>
          </div>

          <div>
            <input type="checkbox" id="specify-members-field" class="l-generator-togglecheck is-hidden">
            <label class="l-generator-toggle" for="specify-members-field">
              <h4 class="l-generator-subtitle l-generator-subtitle--toggleable">{{ $t("generate.specify_members") }}</h4>
            </label>
            <div class="l-generator-field l-generator-field--toggle">
              <CustomMembers />
            </div>
          </div>
        </div>
        <div class="generate-code pure-u-1 pure-u-md-1-2">
          <CodeViewer :code="getCode()" :title="$t('generate.w3c_manifest')" :suggestions="suggestions" :suggestionsTotal="suggestionsTotal"
            :warnings="warnings" :warningsTotal="warningsTotal">
            <nuxt-link :to="$i18n.path('serviceworker')" class="pwa-button pwa-button--simple pwa-button--brand pwa-button--header" @click=" $awa( { 'referrerUri': 'https://preview.pwabuilder.com/generator-nextStep-trigger' })">
              {{ $t("serviceworker.next_step") }}
            </nuxt-link>
          </CodeViewer>
        </div>
      </div>
    </div>

    <div class="l-generator-buttons l-generator-buttons--centered">
      <nuxt-link :to="$i18n.path('serviceworker')" class="pwa-button" @click=" $awa( { 'referrerUri': 'https://preview.pwabuilder.com/generator-nextStep-trigger' })">
        {{ $t("generate.next_step") }}
      </nuxt-link>
    </div>

    <StartOver />
  </div>

  <div v-if="!manifest$">
    <div class="l-generator-step l-generator-step--big"></div>
  </div>
  <TwoWays/>
</section>
</template>

<script lang="ts">
import Vue from 'vue';
import Component from 'nuxt-class-component';
import { Action, State, Getter, namespace } from 'vuex-class';

import GeneratorMenu from '~/components/GeneratorMenu.vue';
import TwoWays from '~/components/TwoWays.vue';
import Modal from '~/components/Modal.vue';
import CodeViewer from '~/components/CodeViewer.vue';
import RelatedApplications from '~/components/RelatedApplications.vue';
import CustomMembers from '~/components/CustomMembers.vue';
import StartOver from '~/components/StartOver.vue';
import ColorSelector from '~/components/ColorSelector.vue';

import * as generator from '~/store/modules/generator';

const GeneratorState = namespace(generator.name, State);
const GeneratorActions = namespace(generator.name, Action);
const GeneratorGetters = namespace(generator.name, Getter);

@Component({
  components: {
    TwoWays,
    GeneratorMenu,
    RelatedApplications,
    CustomMembers,
    ColorSelector,
    CodeViewer,
    StartOver,
    Modal
  }
})
export default class extends Vue {
  public manifest$: generator.Manifest | null = null;
  public newIconSrc = '';
  public iconCheckMissing = true;
  private iconFile: File | null = null;
  public error: string | null = null;

  @GeneratorState manifest: generator.Manifest;
  @GeneratorState members: generator.CustomMember[];
  @GeneratorState icons: generator.Icon[];
  @GeneratorState suggestions: string[];
  @GeneratorState warnings: string[];

  @Getter orientationsNames: string[];
  @Getter languagesNames: string[];
  @Getter displaysNames: string[];

  @GeneratorActions removeIcon;
  @GeneratorActions addIconFromUrl;
  @GeneratorActions updateManifest;
  @GeneratorActions uploadIcon;
  @GeneratorActions generateMissingImages;

  @GeneratorGetters suggestionsTotal;
  @GeneratorGetters warningsTotal;

  public created(): void {
    if (!this.manifest) {
      this.$router.push({
        path: this.$i18n.path('')
      });

      return;
    }

    this.manifest$ = { ...this.manifest };
  }

  public onChangeSimpleInput(): void {
    try {
      this.updateManifest(this.manifest$);
      this.manifest$ = { ...this.manifest };
    } catch (e) {
      this.error = e;
    }
  }

  public onClickRemoveIcon(icon: generator.Icon): void {
    this.removeIcon(icon);
  }

  public onClickAddIcon(): void {
    try {
      this.addIconFromUrl(this.newIconSrc);
    } catch (e) {
      this.error = e;
    }
  }

  public onFileIconChange(e: Event): void {
    const target = e.target as HTMLInputElement;

    if (!target.files) {
      return;
    }

    this.iconFile = target.files[0];
  }

  private getIcons(): string {
    let icons = this.icons.map(icon => {
      return `
        {
            "src": "${icon.src.includes('data:image') ? '[Embedded]' : icon.src}",
            "sizes": "${icon.sizes}"
        }`;
    });
    return icons.toString();
  }

  private getCustomMembers(): string {
    if (this.members.length < 1) {
      return '';
    }

    let membersString = `,
    `;
    this.members.forEach((member, i) => {
      if (i === this.members.length - 1) {
        membersString += `"${member.name}" : "${member.value}"`;
      } else {
        membersString += `"${member.name}" : "${member.value}",
    `;
      }
    });
    return membersString;
  }

  private getManifestProperties(): string {
    let manifest = '';
      for (let property in this.manifest) {
        if (property !== 'icons') {
          manifest += `"${property}" : "${this.manifest[property]}",
    `;
        }
      }
      manifest += `"icons" : [${this.getIcons()}
    ]`;
      manifest += this.getCustomMembers();
    return `{${manifest}}`;
  }

  public getCode(): string | null {
    return this.manifest ? this.getManifestProperties() : null;
  }

  public onClickUploadIcon(): void {
    (this.$refs.iconsModal as Modal).show();
  }

  public async onSubmitIconModal(): Promise<void> {
    const $iconsModal = this.$refs.iconsModal as Modal;

    if (!this.iconFile) {
      return;
    }

    $iconsModal.showLoading();

    if (this.iconCheckMissing) {
      await this.generateMissingImages(this.iconFile);
    } else {
      await this.uploadIcon(this.iconFile);
    }

    $iconsModal.hide();
    $iconsModal.hideLoading();
    this.iconFile = null;
  }

  public onCancelIconModal(): void {
    this.iconFile = null;
  }
}
</script>

<style lang="scss" scoped>
@import "~assets/scss/base/variables";

.generate {
  &-code {
    margin-top: -2rem;
  }
}
</style>