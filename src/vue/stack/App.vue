<template>
    <div>
        <CustomView v-if="isCustomRoute"/>
        <DataManager v-else>
            <WindowObserver>
                <FeedbacksManager>
                    <LanguageManager>
                        <LocationManager>
                            <ModalManager>
                                <Resume/>
                            </ModalManager>
                        </LocationManager>
                    </LanguageManager>
                </FeedbacksManager>
            </WindowObserver>
        </DataManager>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import DataManager from "/src/vue/stack/DataManager.vue"
import LanguageManager from "/src/vue/stack/LanguageManager.vue"
import FeedbacksManager from "/src/vue/stack/FeedbacksManager.vue"
import LocationManager from "/src/vue/stack/LocationManager.vue"
import ModalManager from "/src/vue/stack/ModalManager.vue"
import WindowObserver from "/src/vue/stack/WindowObserver.vue"
import Resume from "/src/vue/stack/Resume.vue"
import CustomView from "/src/vue/custom/CustomView.vue"

const isCustomRoute = ref(false)

onMounted(() => {
    checkRoute()
    window.addEventListener('popstate', checkRoute)
})

const checkRoute = () => {
    // URL'den path'i al
    const path = window.location.pathname
    console.log(path)
    
    // /sdc/{id} pattern kontrolü
    const sdcPattern = /^\/sdc\/([^\/]+)$/
    const match = path.match(sdcPattern)
    
    if (match) {
        // ID'yi al ve CustomView'a gönder
        isCustomRoute.value = true
    } else {
        isCustomRoute.value = false
    }
}
</script>

<style lang="scss" scoped>
</style>