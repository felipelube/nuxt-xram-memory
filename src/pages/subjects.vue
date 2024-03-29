<template>
  <section class="Page SubjectsPage">
    <template v-if="hasData">
      <h1 class="offscreen">
        Assuntos
      </h1>
      <section
        v-if="hasFeaturedSubjects"
        class="SubjectsPage__Featured"
      >
        <header>
          <Microtext
            tag="h2"
            arrow="down"
          >
            Em destaque
          </Microtext>
        </header>
        <div class="SubjectsList">
          <Card
            v-for="subject in featuredSubjects"
            :key="subject.slug"
            class="SubjectCard"
            :item-link="linkFor(subject)"
            :label="labelFor(subject)"
          >
            <h3 slot="title">
              {{ titleFor(subject) }}
            </h3>
            <Microtext slot="label">
              {{ labelFor(subject) }}
            </Microtext>
            <img
              slot="image"
              :src="imageFor(subject)"
            >
          </Card>
        </div>
      </section>
      <section class="AllSubjects">
        <SubjectPicker
          :initial-subjects="initialSubjects"
          :initials="subjectInitials"
        />
      </section>
      <section
        v-if="showTagCloud"
        class="SubjectsPage__TagCloud"
      >
        <header>
          <Microtext
            tag="h2"
            arrow="down"
          >
            Nuvem de palavras-chave
          </Microtext>
        </header>
        <HomeTagCloud
          :size-delta="32"
          :aggregations="tagCloudAggregations"
        />
      </section>
    </template>
    <template v-else>
      <section>
        <header>
          <Microtext
            tag="h2"
            arrow="down"
          >
            Sem dados
          </Microtext>
        </header>
        <main>
          <p>Não existem assuntos para exibir no momento, por-favor, volte mais tarde.</p>
        </main>
      </section>
    </template>
  </section>
</template>

<script>
import HomeTagCloud from '~/components/home/HomeTagCloud'
import Microtext from '~/components/common/Microtext'
import SubjectPicker from '~/components/nav/SubjectPicker'
import Card from '~/components/common/Card'
import { TAGCLOUD_NUM_KEYWORDS } from '~/config/constants'

export default {
  name: 'SubjectsPage',
  components: {
    Microtext,
    HomeTagCloud,
    Card,
    SubjectPicker
  },
  async asyncData ({ $api: { Subjects, Keywords } }) {
    // REFATORAR para usar Promise.all
    let featuredSubjects
    let subjectInitials
    let initialSubjects
    let firstSelectedInitial
    let tagCloudAggregations
    try {
      subjectInitials = await Subjects.getInitials()
      firstSelectedInitial = subjectInitials[0] || ''
      if (firstSelectedInitial) {
        initialSubjects = await Subjects.getByInitial(firstSelectedInitial)
      } else {
        initialSubjects = []
      }
    } catch (e) {
      subjectInitials = []
      initialSubjects = []
    }

    try {
      tagCloudAggregations = await Keywords.all(TAGCLOUD_NUM_KEYWORDS)
    } catch {
      tagCloudAggregations = []
    }

    try {
      featuredSubjects = await Subjects.getFeatured()
    } catch {
      featuredSubjects = []
    }

    return {
      featuredSubjects,
      subjectInitials,
      initialSubjects,
      firstSelectedInitial,
      tagCloudAggregations
    }
  },
  data () {
    return {
      featuredSubjects: [],
      subjectInitials: [],
      initialSubjects: [],
      tagCloudAggregations: []
    }
  },
  head: {
    title: 'xraM-Memory - Assuntos'
  },
  computed: {
    hasData () {
      return this.subjectInitials.length || this.tagCloudAggregations.length
    },
    showTagCloud () {
      return this.tagCloudAggregations.length
    },
    hasFeaturedSubjects () {
      return this.featuredSubjects.length > 0
    }
  },
  methods: {
    labelFor (item) {
      return `${item.items_count} ${item.items_count > 1 ? 'itens' : 'item'}`
    },
    imageFor (item) {
      return this.$utils.getMediaUrl(item.cover)
    },
    titleFor (item) {
      return item.name
    },
    linkFor (item) {
      return {
        name: 'subject-slug',
        params: {
          slug: item.slug
        }
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
.SubjectsPage > section
  max-width: $max-width
  margin: 4rem auto

.SubjectsPage > section.SubjectsPage__TagCloud

.SubjectsPage > section.SubjectsPage__TagCloud > header
  max-width: $max-width
  margin: 0 auto

.SubjectsPage > section:first-of-type
  margin-top: 0

.SubjectsList
  grid-auto-flow: row
  grid-template-columns: repeat(auto-fill, 250px)
  width: 100%
  grid-column-gap: 20px
  display: grid
  grid-row-gap: 20px
  justify-content: space-evenly

.SubjectCard
  transition: all 0.25s ease
  min-height: 275px

.SubjectCard img
  margin-top: auto
  position: relative
  display: inline-block
  filter: grayscale($grayscale-image)

.SubjectCard a:active img, .SubjectCard a:focus img, .SubjectCard a:hover img
  filter: none

h3, .microtext
  order: -1

.Card p.microtext
  color: #777474
  font-size: 12px

h3
  font-family: $small-caps
  margin-top: 0.25rem
  margin: 1rem
  font-size: 22px
  line-height: 25px
  text-align: center
  color: #000000
  word-break: break-word

footer
  text-align: right

</style>
