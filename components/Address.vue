<template>
  <div
    class="
      relative
      flex
      items-top
      justify-center
      min-h-screen
      bg-gray-100
      sm:items-center sm:pt-0
    "
  >
    <div
      class="max-w-4xl mx-auto sm:px-6 lg:px-8 bg-gray-800 h-96 overflow-auto"
    >
      <div class="mt-8 sm:rounded-lg p-6">
        <div class="mt-10 sm:mt-0">
          <div class="grid grid-cols-3 gap-6">
            <div class="col-span-1">
              <div class="px-4 sm:px-0">
                <div
                  class="
                    w-32
                    h-6
                    border-b
                    bg-gray-800
                    border-gray-300
                    relative
                    cursor-pointer
                  "
                  @click="showAddressType = !showAddressType"
                >
                  <span class="text-white">{{ selectedAddressType }}</span>
                  <ChevronDownIcon
                    class="inline text-white float-right w-4 w-4"
                  />
                  <div
                    v-if="showAddressType"
                    class="absolute top-9 left-0 bg-gray-500 pl-3 pr-6 py-2"
                  >
                    <div
                      v-for="(addressType, i) in addressTypes"
                      :key="i"
                      class="
                        text-white
                        border-b border-gray-400
                        cursor-pointer
                        my-1
                      "
                      @click="selectedAddressType = addressTypes[i]"
                    >
                      {{ addressType }}
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="col-span-2">
              <div class="px-4 flex items-center">
                <QuestionMarkCircleIcon
                  class="inline text-white rounded-full w-4 h-4 cursor-pointer"
                />
                <div class="w-64 bg-gray-800 ml-2">
                  <input
                    v-if="!addressString"
                    id="addressInput"
                    ref="searchAddressField"
                    v-model="address"
                    type="text"
                    class="
                      w-full
                      bg-gray-800
                      text-white
                      focus:outline-none
                      border-0
                    "
                    placeholder="Address"
                  />
                  <div
                    v-if="!address.length > 0 && !addressString"
                    class="w-full h-0.5 bg-gray-100"
                  ></div>
                  <div
                    v-if="address.length > 0 || addressString"
                    class="w-full"
                  >
                    <div class="text-white">{{ addressString }}</div>
                    <div class="flex items-center justify-between">
                      <div class="w-1/3 h-0.5 bg-gray-100"></div>
                      <div class="text-white">ADDRESS</div>
                      <div class="w-1/3 h-0.5 bg-gray-100"></div>
                    </div>
                  </div>
                </div>

                <div
                  v-if="showAddressNumberField"
                  class="w-24 bg-gray-800 ml-2"
                >
                  <input
                    v-model="addressNumber"
                    type="text"
                    class="
                      w-full
                      bg-gray-800
                      text-white
                      focus:outline-none
                      border-0
                    "
                    tabindex="0"
                    @blur="updateAddressString"
                  />
                  <div
                    v-if="!addressNumber.length > 0"
                    class="w-full h-0.5 bg-gray-100"
                  ></ChevronDownIcon>
                  <div v-if="addressNumber.length > 0" class="w-full">
                    <div class="flex items-center justify-between">
                      <div class="w-1/3 h-0.5 bg-gray-100"></div>
                      <div class="text-white">NO</div>
                      <div class="w-1/3 h-0.5 bg-gray-100"></div>
                    </div>
                  </div>
                </div>

                <div
                  v-if="showClose"
                  class="
                    bg-gray-400
                    rounded-full
                    h-5
                    w-5
                    flex
                    items-center
                    justify-center
                    cursor-pointer
                    ml-2
                  "
                  @mousedown="closeIconAction"
                >
                  <XIcon class="inline text-white rounded-full w-3 h-3" />
                </div>

                <PlusIcon
                  v-if="showAddIcon"
                  class="
                    inline
                    text-white
                    rounded-full
                    w-6
                    h-6
                    ml-2
                    cursor-pointer
                  "
                  @click="setShowAddressNumberField"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  ChevronDownIcon,
  XIcon,
  PlusIcon,
  QuestionMarkCircleIcon,
} from '@vue-hero-icons/outline'

export default {
  components: {
    ChevronDownIcon,
    XIcon,
    PlusIcon,
    QuestionMarkCircleIcon,
  },
  data() {
    return {
      addressTypes: ['Residential', 'Domicile', 'Legal', 'Operational'],
      showAddressType: false,
      selectedAddressType: '',
      address: '',
      addressString: '',
      addressNumber: '',
      showAddressNumberField: false,
      showAddIcon: false,
      apiKey: '',
      autoCompleteObject: null,
    }
  },
  head() {
    return {
      script: [
        {
          src: `https://maps.googleapis.com/maps/api/js?key=AIzaSyBNDxFPZ6ZtvbIT7NxRaujPDunbFnS0sIk&libraries=places`,
        },
      ],
    }
  },

  computed: {
    showClose() {
      return this.address.length > 0 || this.addressNumber
    },
  },
  mounted() {
    this.selectedAddressType = this.addressTypes[0]
    this.apiKey = this.$config.GOOGLE_MAPS_API_KEY
    const addressInput = this.$refs.searchAddressField
    // eslint-disable-next-line no-new
    let autocomplete
    // eslint-disable-next-line prefer-const
    autocomplete = new window.google.maps.places.Autocomplete(addressInput)

    const _this = this
    window.google.maps.event.addListener(
      autocomplete,
      'place_changed',
      function (e) {
        _this.addressString = autocomplete.getPlace().formatted_address
        _this.address = ''
        _this.showAddIcon = true
      }
    )
  },
  methods: {
    setShowAddressNumberField() {
      this.showAddressNumberField = true
      this.showAddIcon = false
    },
    closeIconAction() {
      this.address = ''
      this.addressNumber = ''
    },
    updateAddressString() {
      if (!this.addressNumber) return
      this.addressString = this.addressNumber + ', ' + this.addressString
      this.addressNumber = ''
      this.showAddressNumberField = false
    },
  },
}
</script>
