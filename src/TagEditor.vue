<template>
  <div class="tageditor form-horizontal">
    <div class="form-group tag-line" v-for="(ebenenTags, etKey) in tags" :key="'teet' + etKey">
      <label class="col-sm-3 control-label">
        <select class="tagebene w100" v-model="ebenenTags.e" :disabled="!edit">
          <option :value="0">Ebene auswählen (Löschen)</option>
          <option v-for="tagebene in tagsData.data.baseCache.tagebenenList" :key="'teet' + etKey + 'tesel' + tagebene.pk"
                  :value="tagebene.pk"
                  :disabled="(ebenenTags.e !== tagebene.pk) && ebeneVorhanden(tagebene.pk)">
                    {{ tagebene.t }}
          </option>
        </select>
      </label>
      <div class="col-sm-9">
        <div class="form-control-static reihung-tags" v-if="ebenenTags.e > 0">
          <div class="r-tag-familie r-tag-familie-pchilds">
            <TagEditorTags :ebenenPK="ebenenTags.e" :generation="0" :tags="ebenenTags.tags" :parents="[]" :edit="edit" :tagsData="tagsData" @changed="changed" />
          </div>
          <div class="iblock prel" v-if="getValOfSubProp(tagsData.data.baseCache.tagebenenObj, ebenenTags.e + '.presets.length') > 0">
            <button class="ant-ctag" :disabled="tagsData.data.loadingPresets" @click="togglePreset(etKey)"><span class="glyphicon glyphicon-star" aria-hidden="true"></span></button>
            <div class="tags seltags open" v-if="!tagsData.data.loadingPresets && showPresets[etKey]">
              <button class="pretagsbtn" :title="(pIndex + 1) + '. ' + preset.b  + ' - ' + preset.tokenText"
                v-for="(preset, pIndex) in getFilteredPresets(ebenenTags)"
                :key="'psbtn' + pIndex"
                @click="addPreset(ebenenTags, preset)" v-on:blur="selPresetBlur">{{ (pIndex + 1) }}. <b>{{ preset.b }}</b> - {{ preset.tokenText }}</button>
            </div>
          </div>
        </div>
        <p class="form-control-static" v-else><b>Ebene auswählen!</b></p>
      </div>
    </div>
    <div class="form-group add-tag-line-line" v-if="edit">
      <div class="col-sm-3"><button class="add-tag-line" @click="addEbene"><span class="glyphicon glyphicon-plus" aria-hidden="true"></span> Tag-Ebene</button></div>
    </div>
  </div>
</template>

<script>
/* global $ _ */
import TagEditorTags from './TagEditorTags'
// var _ = require('lodash')

export default {
  name: 'TagEditor',
  props: ['tagsData', 'tags', 'mode'],
  data () {
    return {
      showPresets: {}
    }
  },
  mounted () {
  },
  watch: {
  },
  methods: {
    ebeneVorhanden (eId) {
      let vorhanden = false
      this.tags.some(function (aEbene) {
        if (aEbene.e === eId) {
          vorhanden = true
          return true
        }
      }, this)
      return vorhanden
    },
    addEbene () {
      this.tags.push({'e': 0, 'tags': []})
    },
    togglePreset (etKey) {
      this.$set(this.showPresets, etKey, !this.showPresets[etKey])
      this.$nextTick(() => $('.pretagsbtn:first-child').focus())
    },
    selPresetBlur: _.debounce(function () {
      if (document.activeElement.className.indexOf('pretagsbtn')) {
        this.showPresets = {}
      }
    }, 20),
    addPreset (ebenenTags, preset) {
      preset.tags.forEach(function (val) {
        ebenenTags.tags.push(_.cloneDeep(val))
      }, this)
      this.showPresets = {}
    },
    getFilteredPresets (ebenenTags) {
      let presetList = []
      this.tagsData.data.presetsCache.presetsList.forEach(aPreset => {
        if ((aPreset.tf.length > 0) && (!aPreset.ze || (aPreset.ze && aPreset.ze.indexOf(ebenenTags.e) > -1))) {
          presetList.push(aPreset)
        }
      })
      return presetList
    },
    getValOfSubProp (obj, propertys) {    // Gibt Wert eines Property eines verschachtelten Objekts zurück
      return this.hasSubProp(obj, propertys, true)
    },
    hasSubProp (obj, propertys, retVal = false) {   // Ermitten ob Property in einem verschachtelten Objekt existiert
      var out = null
      if ((typeof propertys === 'string') && (propertys !== null) && propertys.length > 0) {
        var aObj = obj
        propertys.split('.').some(function (property) {
          if ((typeof aObj === 'object') && (aObj !== null)) {
            if (aObj.hasOwnProperty(property)) {
              out = true
              aObj = aObj[property]
            } else {
              out = null
              return true
            }
          } else {
            out = null
            return true
          }
        })
      }
      return ((retVal) ? ((out) ? aObj : null) : out)
    },
    changed () {
      this.$emit('changed')
    }
  },
  computed: {
    edit () {
      return this.mode === 'edit'
    }
  },
  components: {
    TagEditorTags
  }
}
</script>

<style scoped>
</style>
