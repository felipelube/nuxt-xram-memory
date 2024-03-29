<template>
  <div class="NewsInfo FieldList">
    <div
      v-if="teaser"
      class="FieldList__Field"
    >
      <h2>Resumo da notícia</h2>
      <p class="NewsInfo__Teaser">
        {{ teaser }}
      </p>
    </div>
    <div
      v-if="url"
      class="FieldList__Field"
    >
      <h2>Endereço original</h2>
      <p>
        <a
          :href="url.url"
          target="_blank"
        >{{ url.url }}</a>
      </p>
    </div>
    <div
      v-if="archivedUrl"
      class="FieldList__Field"
    >
      <h2>Versão arquivada</h2>
      <p>
        <a
          :href="archivedUrl.url"
          target="_blank"
        >{{ archivedUrl.url }}</a>
      </p>
    </div>
    <div
      v-if="newspaper"
      class="FieldList__Field"
    >
      <h2>Site / veículo</h2>
      <p>
        <a
          class="NewsInfo__Newspaper"
          :href="newspaper.url"
          target="_blank"
        >
          <img
            v-if="newspaperIcon"
            :src="newspaperIcon"
            alt
          >
          {{ newspaper.title }}
        </a>
      </p>
    </div>
    <div
      v-if="publishedDate"
      class="FieldList__Field"
    >
      <h2>Data de publicação</h2>
      <time>{{ publishedDate }}</time>
    </div>
    <div
      v-if="pdfCaptures.length"
      class="FieldList__Field FieldList__Field--centered NewsInfo--pdf-captures"
    >
      <h2>Capturas de página</h2>
      <nuxt-link
        v-for="capture in pdfCaptures"
        :key="capture.url"
        class="NewsInfo__PDFCapture"
        :to="{
          name: 'document-document_id',
          params: {
            document_id: capture.document_id
          },
        }"
      >
        <v-img
          :key="capture.url"
          :src="thumbnailForDocument(capture.document_id)"
          aspect-ratio="1"
          class="ImageCapture"
          :alt="capture.title"
          :title="capture.title"
          :contain="false"
          @error="changeImagePlaceholder"
        >
          <template
            #placeholder
            class="image-slot"
          >
            <div class="image-slot">
              <v-icon>download</v-icon>
              <Microtext>{{ imagePlaceholder }}</Microtext>
            </div>
          </template>
        </v-img>
        <Microtext tag="p">
          {{ capture.title }}
        </Microtext>
      </nuxt-link>
    </div>
    <div
      v-if="subjects.length"
      class="FieldList__Field"
    >
      <h2>Assuntos</h2>
      <ul class="taxonomy-list">
        <li
          v-for="subject in subjects"
          :key="subject.slug"
        >
          <nuxt-link :to="{name:'subject-slug', params:{ slug: subject.slug} }">
            {{ subject.name }}
          </nuxt-link>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Microtext from '@/components/common/Microtext'
const smartTruncate = require('smart-truncate')
const humanSize = require('human-size')
const dayJs = require('dayjs')
export default {
  name: 'NewsInfo',
  components: {
    Microtext
  },
  props: {
    newsItem: {
      type: Object,
      default: function () {
        return {}
      }
    }
  },
  data () {
    return {
      documents: [],
      imagePlaceholder: 'Carregando...'
    }
  },
  computed: {
    teaser () {
      return this.newsItem.teaser
    },
    subjects () {
      try {
        return (
          this.newsItem.subjects &&
          this.newsItem.subjects.length &&
          this.newsItem.subjects
        )
      } catch {
        return []
      }
    },
    pdfCaptures () {
      try {
        return (
          this.newsItem.pdf_captures &&
          this.newsItem.pdf_captures.map(capture => {
            return {
              document_id: capture.pdf_document.document_id,
              title: this.captureNameAndSize(capture)
            }
          })
        )
      } catch {
        return []
      }
    },
    publishedDate () {
      try {
        const [time, date] = this.newsItem.published_date && [
          new Date(this.newsItem.published_date).toLocaleTimeString(),
          new Date(this.newsItem.published_date).toLocaleDateString()
        ]
        return `${date} ${time}`
      } catch {
        return ''
      }
    },
    newspaper () {
      try {
        return (
          this.newsItem.newspaper &&
          this.newsItem.newspaper.url &&
          this.newsItem.newspaper.title && {
            url: this.newsItem.newspaper.url,
            title: this.newsItem.newspaper.title
          }
        )
      } catch {
        return ''
      }
    },
    newspaperIcon () {
      try {
        return this.$utils.getMediaUrl(this.newsItem.newspaper.favicon_logo)
      } catch (e) {
        return ''
      }
    },
    url () {
      try {
        return {
          url: this.newsItem.url,
          title: smartTruncate(this.newsItem.url, 120, { position: 20 })
        }
      } catch {
        return ''
      }
    },
    archivedUrl () {
      try {
        if (this.newsItem.archived_news_url) {
          return {
            url: this.newsItem.archived_news_url,
            title: smartTruncate(this.newsItem.archived_news_url, 120, {
              position: 20
            })
          }
        }
        return ''
      } catch {
        return ''
      }
    }
  },
  watch: {
    pdfCaptures: {
      immediate: true,
      async handler () {
        try {
          this.documents = await this.getDocumentsInfo()
        } catch {
          this.documents = []
        }
      }
    }
  },
  methods: {
    changeImagePlaceholder () {
      this.imagePlaceholder = 'Falha ao carregar a imagem.'
    },
    captureNameAndSize (capture) {
      const title = {}
      try {
        const dateTime = dayJs(capture.pdf_capture_date)
        if (!dateTime.isValid()) {
          throw new Error()
        }
        title.date = dateTime.toDate().toLocaleDateString()
      } catch {
        // não há data válida, ignore
      }

      try {
        const size = humanSize(parseInt(capture.pdf_document.size, 10))
        title.size = size
      } catch {
        // não há tamanho válido, ignore
      }
      return `${title.date || 'Captura em PDF'} ${
        title.size ? `(${title.size})` : ''
      }`
    },
    async getDocumentsInfo () {
      return Promise.all(
        this.pdfCaptures.map(capture => {
          return this.$api.Documents.getById(capture.document_id)
        })
      )
    },
    thumbnailForDocument (documentId) {
      try {
        return this.$utils.getMediaUrl(
          this.documents.find(document => document.document_id === documentId)
            .thumbnails.document_thumbnail
        )
      } catch {
        return ''
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
.NewsInfo__PDFCapture
.ImageCapture
  width: 250px
  height: 250px
  display: inline-block

.ImageCapture
  padding: 2px

.image-slot
  display: flex
  align-items: center
  flex-direction: column
  justify-content: center
  height: 100%

.image-slot.error > i
  color: $error-color

.image-slot.error > p
  color: #900

.NewsInfo--pdf-captures a
  border: solid 1px #efefef
  margin: 6px
  overflow: hidden
  transition: border-color 0.25s ease

.NewsInfo--pdf-captures p
  display: block
  text-align: center

.NewsInfo--pdf-captures a
  &:active
  &:focus
  &:hover
    border-color: $link-color

.NewsInfo__Newspaper
  display: inline-flex

.NewsInfo__Newspaper > img
  padding-right: 5px

ul.taxonomy-list
  margin: 0
  padding: 0
  display: flex
  flex-wrap: wrap
  align-items: center
  justify-content: flex-start

ul.taxonomy-list > li
  display: inline-block
  border: 1px solid transparent
  line-height: 1.2rem
  color: #333333
  cursor: pointer
  user-select: none
  transition: all 0.3s ease
  padding: 5px
  @media only screen and (min-width: $tablet)
    margin: 0.5rem

p.NewsInfo__Teaser
  line-height: 1.5
  text-align: left

</style>
