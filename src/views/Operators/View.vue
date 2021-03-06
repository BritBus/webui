<template>
  <div v-if="error">
    {{ error }}
  </div>
  <div v-if="loading">Loading...</div>
  <div v-else class="h-full">
    <PageTitle class="pb-0 md:pb-0 lg:pb-0">
      {{ this.operator.PrimaryName }}
    </PageTitle>

    <NavTabBar
      class="mb-2"
      :tabs="tabs"
      :currentTab="currentTab"
      :changeTab="changeTab"
    />

    <OperatorOverview :operator="this.operator" v-bind:class="{ hidden: this.currentTab != 'overview' }" />

    <OperatorServices :operator="this.operator" v-bind:class="{ hidden: this.currentTab != 'services' }" />

    <Card v-bind:class="{ hidden: this.currentTab != 'stats' }">
      Stats
    </Card>
  </div>
</template>

<script>
import PageTitle from '@/components/PageTitle.vue'
import Card from '@/components/Card.vue'
import NavTabBar from '@/components/NavTabBar.vue'
import OperatorOverview from '@/components/Operators/Overview.vue'
import OperatorServices from '@/components/Operators/Services.vue'
import axios from 'axios'
import API from '@/API'

export default {
  name: 'OperatorsView',
  data () {
    return {
      operator: null,
      loading: true,
      error: null,

      defaultTab: 'overview',
      currentTab: null,
      tabs: [
        {
          id: "overview",
          name: "Overview"
        },
        {
          id: "services",
          name: "Services"
        },
        {
          id: "stats",
          name: "Statistics"
        }
      ]
    }
  },
  components: {
    PageTitle,
    Card,
    NavTabBar,
    OperatorOverview,
    OperatorServices
  },
  methods: {
    changeTab(newTab) {
      this.$router.push({ name: this.$route.name, params: {id:this.$route.params.id}, query: {tab: newTab} })
    },
    getOperator() {
      axios
      .get(`${API.URL}/core/operators/${this.$route.params.id}`)
      .then(response => {
        this.operator = response.data
      })
      .catch(error => {
        console.log(error)
        this.error = error
      })
      .finally(() => this.loading = false)
    },
  },
  mounted () {
    this.getOperator()
    
    if (this.$route.query.tab !== undefined) {
      this.currentTab = this.$route.query.tab
    } else {
      this.currentTab = this.defaultTab
    }
  },
  created() {
    this.$watch(
      () => this.$route.query,
      (toQuery, previousQuery) => {
        if (toQuery.tab == previousQuery.tab) {
          return
        }

        if (toQuery.tab !== undefined) {
          this.currentTab = toQuery.tab
        } else {
          this.currentTab = this.defaultTab
        }
      }
    )
  },
}
</script>