<script setup lang="ts">
import { SbcHeaderMain, SbcFooter, SbcBreadcrumb } from '#components'
const { locale } = useI18n()
const localePath = useLocalePath()
// const user = useCurrentUser()
// const router = useRouter()
// const route = useRoute()
const i18nHead = useLocaleHead({
  addDirAttribute: true,
  addSeoAttributes: true
})

useHead({
  htmlAttrs: {
    lang: () => i18nHead.value.htmlAttrs!.lang,
    dir: () => i18nHead.value.htmlAttrs!.dir
  }
})

// template ref for header component
const headerRef = ref<InstanceType<typeof SbcHeaderMain> | null>(null)
const footerRef = ref<InstanceType<typeof SbcFooter> | null>(null)
const breadcrumbRef = ref<InstanceType<typeof SbcBreadcrumb> | null>(null)
const { height: mainHeaderHeight } = useElementSize(headerRef)
const { height: footerHeight } = useElementSize(footerRef)
const { height: breadcrumbHeight } = useElementSize(breadcrumbRef)

// fetch content files using composable from nuxt-content https://content.nuxt.com/composables/fetch-content-navigation
const { data: docNavItems } = await useAsyncData(
  'content-navigation',
  () => fetchContentNavigation(
    queryContent('products') // pass custom query to fetchContentNavigation
      .where({ _locale: locale.value, _extension: { $eq: 'md' } }) // only return md files that match current locale
      .sort({ _dir: 1 }) // sort alphabetically
  ), {
    watch: [locale] // fetch new values whenever the locale changes
  }
)

// dashboard page sub nav items
const dashboardNavItems = [
  {
    label: 'Dashboard Nav Item',
    defaultOpen: true,
    children: [
      {
        label: 'Child 1',
        to: localePath('/sbc/dashboard')
      },
      {
        label: 'Child 2',
        to: localePath('/sbc/dashboard')
      }
    ]
  }
]

// provide nav items to use in docs/dashboard layouts
provide('docNavItems', docNavItems)
provide('dashNavItems', dashboardNavItems)
// provide element heights to offset headers/asides
provide('mainHeaderHeight', mainHeaderHeight)
provide('footerHeight', footerHeight)
provide('breadcrumbHeight', breadcrumbHeight)

// watch(user, async (currentUser, previousUser) => {
//   // redirect user home if they sign in from the login page without a redirect
//   if (currentUser && !previousUser && route.path.includes('/sbc/auth/login')) {
//     await navigateTo(localePath('/'))
//   }

//   // redirect the user if they are logged in but were rejected because the user wasn't ready yet
//   if (currentUser && typeof route.query.redirect === 'string') {
//     return router.push(route.query.redirect)
//   }
// })
</script>
<template>
  <div
    class="flex min-h-screen flex-col bg-bcGovColor-gray1 dark:bg-bcGovGray-900"
  >
    <SbcHeaderMain ref="headerRef" class="sticky inset-x-0 top-0 z-50" />
    <SbcMobileNav
      :accordian-nav-items="
        $route.path.includes('products') ? createContentNav(docNavItems!) :
        $route.path.includes('dashboard') ? dashboardNavItems : undefined
      "
    />
    <SbcBreadcrumb
      v-if="!$route.path.includes('oas')"
      ref="breadcrumbRef"
      class="sticky inset-x-0 top-16 z-40 hidden md:block"
    />
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
    <SbcFooter ref="footerRef" class="z-50 mt-auto" />
  </div>
</template>
<style>
  .light-mode {
    --scalar-color-1: #212529 !important;
    --scalar-color-2: #212529 !important;
    --scalar-color-3: #212529 !important;
    --scalar-color-accent: #1669bb !important;
    --scalar-background-1: #f1f3f5 !important;
    --scalar-background-2: #e9ecef !important;
    --scalar-background-3: #dee2e6 !important;
    --scalar-background-accent: #5369d20f !important;
    --scalar-border-color: rgba(0, 0, 0, 0.08) !important;
  }
  .dark-mode {
    --scalar-color-1: rgba(255, 255, 255, 0.81) !important;
    --scalar-color-2: rgba(255, 255, 255, 0.443) !important;
    --scalar-color-3: rgba(255, 255, 255, 0.282) !important;
    --scalar-color-accent: #e0e7ed !important;
    --scalar-background-1: #202020 !important;
    --scalar-background-2: #272727 !important;
    --scalar-background-3: #333333 !important;
    --scalar-background-accent: #8ab4f81f !important;
    --scalar-border-color: rgb(209 213 219 / 0.5) !important;
  }
</style>
