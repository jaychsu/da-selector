<template lang="pug">
  .da-selector(
    ref="container"
    :class="{ 'is-expanded': isPanelVisible_ }"
  )
    .da-selector-placeholder {{ selectedOptionData_.display }}

    .da-selector-search-wrap
      input.da-selector-search(
        ref="searcher"
        type="text"
        :placeholder="selectedOptionData_.display"
      )
      span.da-selector-search-icon

    span.da-selector-triangle

    .da-selector-panel(
      :class="{ hide: !isPanelVisible_ }"
    )
      ul.da-selector-option-list
        li.da-selector-option-item(
          v-for="optionData in optionDataList_"
          :class="{ active: optionData.id === selectedOptionData_.id }"
          @click="() => selectOption(optionData)"
        ) {{ optionData.display }}

    label(
      :for="id"
    ) {{ placeholder }}
    input(
      type="hidden"
      :id="id"
    )
</template>

<script>
  const OptionData = {
    id: '',
    display: '',
  }

  const ID_NO_RESULT = 'no-result'
  const NULL_FN = () => {}

  export default {
    props: {
      id: {
        type: String,
        required: true,
      },
      placeholder: {
        type: String,
        default: '',
      },
      onChange: {
        type: Function,
        default: NULL_FN,
      },

      optionDataList: {
        type: Array,
        required: true,
        default() {
          return [ OptionData ]
        },
      },
      selectedOptionData: {
        type: Object,
        default() {
          return this.optionDataList[0]
        },
      },
      isPanelVisible: {
        type: Boolean,
        default: false,
      },
    },
    data() {
      return {
        optionDataList_: this.optionDataList,
        selectedOptionData_: this.selectedOptionData,
        isPanelVisible_: this.isPanelVisible,
      }
    },
    methods: {
      selectOption(optionData) {
        if (optionData.id === ID_NO_RESULT) return false

        if (!optionData.display) optionData.display = optionData.id

        const prevOptionData = this.selectedOptionData_
        this.onChange(optionData, prevOptionData)
        this.selectedOptionData_ = optionData

        this.togglePanel(false)
      },
      togglePanel(isPanelVisible) {
        if (typeof isPanelVisible === 'undefined') {
          isPanelVisible = !this.isPanelVisible_
        } else {
          isPanelVisible = !!isPanelVisible
        }

        this.isPanelVisible_ = isPanelVisible
      },
      handlePageEvent(event) {
        const targetClass = event.target.className
        const isOutsideComponent = !this.$refs['container'].contains(event.target)
        const needNoResponse =
          targetClass.indexOf('da-selector-option') > -1
          || targetClass.indexOf('da-selector-search') > -1

        if (needNoResponse) {
          // need no response
          return false
        } else if (isOutsideComponent) {
          // outside component
          this.togglePanel(false)
          return false
        } else {
          // toggle panel
          this.togglePanel()
          if (this.isPanelVisible_) {
            // Here is a hack to use `focus` within `setTimeout`
            setTimeout(() => this.$refs['searcher'].focus(), 0)
          }
        }
      },
    },

    mounted() {
      window.addEventListener('click', this.handlePageEvent, false)
    },
    beforeDestroy() {
      window.removeEventListener('click', this.handlePageEvent, false)
    },
  }
</script>

<style src="./da-selector.css"></style>