<template>
    <div class="project-modal-content">
        <template v-if="!isCreating">
            <!-- Title -->
            <h1 class="mb-2 fw-bold"
                v-html="localize(item.locales, 'title')"/>

            <!-- Tags -->
            <div v-if="parsedTags && parsedTags.length"
                 class="tags-wrapper text-3 mt-2 mt-lg-3">
                <span class="d-none d-lg-inline">
                    <i class="fa-solid fa-tag opacity-75"/>
                    {{localizeFromStrings("tags")}}:
                </span>

                <Tags class="mt-1 mt-lg-0" :tags="parsedTags"/>
            </div>

            <!-- Blocks -->
            <template v-for="block in blocks">
                <div v-if="block.visibility" class="project-modal-content-block">
                    <h5 :class="block.titleClass">
                        <i :class="`${block.titleFaIcon}`"/>
                        <span v-html="block.title"/>
                    </h5>

                    <p class="text-3 mb-0" v-html="block.description"/>

                    <SocialLinks v-if="block.links"
                                 :items="block.links"
                                 class="social-links"
                                 size="2"
                                 variant="dark"/>
                </div>
            </template>

            <!-- Create Button -->
            <div class="create-button-wrapper mt-4">
                <button class="btn btn-primary w-100" @click="startCreating">
                    <i class="fa-solid fa-plus me-2"></i>
                    {{ localizeFromStrings("create") }}
                </button>
            </div>
        </template>

        <template v-else>
            <template v-if="!generatedCardId">
                <CardCreator 
                    :formFields="formFields"
                    :isLoading="isLoading"
                    @submit="handleCardSubmit"
                    @cancel="cancelCreating"
                />

                <div v-if="errorMessage" class="alert alert-danger mt-3">
                    <i class="fa-solid fa-circle-exclamation me-2"></i>
                    {{ errorMessage }}
                </div>
            </template>

            <div v-else class="card-result">
                <div class="text-center">
                    <h3 class="mb-4">🎉 Harika! Doğum günü kartın hazır!</h3>
                    
                    <div class="link-container mb-4">
                        <div class="d-flex align-items-center gap-2">
                            <input 
                                type="text" 
                                class="form-control" 
                                :value="`${siteDomain}/sdc/${generatedCardId}`"
                                readonly
                                ref="linkInput"
                            />
                            <button 
                                class="btn btn-primary" 
                                @click="copyLink"
                            >
                                <i class="fa-solid fa-copy"></i>
                            </button>
                        </div>
                        <small v-if="isCopied" class="text-success mt-2 d-block">
                            <i class="fa-solid fa-check me-1"></i>
                            Link kopyalandı!
                        </small>
                    </div>

                    <p class="text-muted">
                        <i class="fa-solid fa-paper-plane me-2"></i>
                        Şimdi bu linki kutlamak istediğin kişiye gönder ve sürprizi bozma! 🎁
                    </p>
                </div>
            </div>
        </template>
    </div>
</template>

<script setup>
import {computed, inject, ref} from "vue"
import Tags from "/src/vue/components/widgets/Tags.vue"
import SocialLinks from "/src/vue/components/widgets/SocialLinks.vue"
import CardCreator from "/src/vue/components/tools/CardCreator.vue"

const props = defineProps({
    /** @type {ArticleItem} **/
    item: {
        type: Object,
        required: false
    },
})

/** @type {Function} */
const localize = inject("localize")

/** @type {Function} */
const localizeFromStrings = inject("localizeFromStrings")

const parsedTags = computed(() => {
    const tags = localize(props.item.locales, "tags")
    if(Array.isArray(tags))
        return tags
    return []
})

const blocks = computed(() => {
    const description = localize(props.item.locales, "description")
    const links = props.item.links

    return [
        {
            visibility: description,
            titleClass: "d-none d-lg-inline-block",
            title: localizeFromStrings("about"),
            titleFaIcon: "fa-solid fa-file",
            description: description,
            links: null
        },
        // {
        //     visibility: links?.length,
        //     titleClass: "",
        //     title: localizeFromStrings("where_to_find"),
        //     titleFaIcon: "fa-solid fa-globe",
        //     description: localizeFromStrings("where_to_find_description")
        //         .replace("{project}", `<b>${localize(props.item.locales, "title")}</b>`),
        //     links: links.map((link) => {return {
        //         href: link.href,
        //         faIcon: link.faIcon,
        //         id: link.stringKey
        //     }})
        // }
    ]
})

const isCreating = ref(false)

const formFields = computed(() => {
    return localize(props.item.locales, "form") || []
})

const isLoading = ref(false)
const generatedCardId = ref(null)
const linkInput = ref(null)
const siteDomain = window.location.origin
const errorMessage = ref('')
const isCopied = ref(false)

const startCreating = () => {
    isCreating.value = true
}

const cancelCreating = () => {
    isCreating.value = false
    generatedCardId.value = null
    errorMessage.value = ''
}

const copyLink = () => {
    if (linkInput.value) {
        linkInput.value.select()
        document.execCommand('copy')
        isCopied.value = true
        setTimeout(() => {
            isCopied.value = false
        }, 2000)
    }
}

const mapFormDataToPayload = (formData) => ({
  type: 'birthday',
  message: formData["Özel mesaj"] || formData["Special Message"] || "",
  recipient: formData["Kutlanan kişinin adı"] || formData["Celebrated Person's Name"] || "",
  sender: formData["Kutlayanın adı"] || formData["Celebrator's Name"] || ""
});

const handleCardSubmit = async (formData) => {
    try {
        isLoading.value = true
        errorMessage.value = ''
        const payload = mapFormDataToPayload(formData)
        // Alanlar boşsa gönderme
        if (!payload.message || !payload.recipient || !payload.sender) {
            errorMessage.value = "Lütfen tüm alanları doldurun.";
            isLoading.value = false;
            return;
        }
        const response = await fetch('https://ne41k5dtad.execute-api.eu-north-1.amazonaws.com/cards', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(payload)
        })

        const data = await response.json()
        if (data.card_id) {
            generatedCardId.value = data.card_id
        } else {
            throw new Error('Kart oluşturulamadı')
        }
    } catch (error) {
        console.error('Error creating card:', error)
        errorMessage.value = error.message || 'Bir hata oluştu. Lütfen tekrar deneyin.'
    } finally {
        isLoading.value = false
    }
}
</script>

<style lang="scss" scoped>
@import "/src/scss/_theming.scss";

div.project-modal-content {
    width: 100%;
    @include media-breakpoint-down($navigation-sidebar-breakpoint) {
        text-align: center;
    }
}

div.tags-wrapper {
    display: flex;

    span {
        display: flex;
        align-items: center;
        justify-content: center;

        white-space: nowrap;
        margin-right: 10px;
        margin-bottom: 3px;

        i {
            margin-right: 4px;
        }
    }

    @include media-breakpoint-down($navigation-sidebar-breakpoint) {
        flex-direction: column;
        span {
            margin-bottom: 8px;
        }
    }
}

div.project-modal-content-block {
    margin-top: 30px;

    h5 {
        font-weight: bold;
    }

    i {
        min-width: 25px;
        margin-right: 5px;
        text-align: center;
        @include media-breakpoint-down(md) {
            min-width: 0;
        }
    }

    div.social-links {
        margin-top: 12px;
    }

    @include media-breakpoint-down($navigation-sidebar-breakpoint) {
        margin-top: 25px;
    }
}

div.create-button-wrapper {
    display: flex;
    justify-content: center;
    
    button {
        min-width: 150px;
    }
}

.card-result {
    .link-container {
        max-width: 600px;
        margin: 0 auto;
    }
}
</style>