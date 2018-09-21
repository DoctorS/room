<template>
  <div id="app">
    <!-- top panel -->
    <div>
      <div class="filter">
        <div class="TypeSearch">
          <div>
            <button class="btns">Добавить запрос</button><!--success-->
          </div>
          <div>
            <button class="btns" @click="showPanel = !showPanel">{{showPanel ? 'Скрыть панель':'Показать панель'}}
            </button>
          </div>
        </div>
      </div>
    </div>
    <!-- / top panel -->

    <div class="panel" v-if="showPanel">
      <div>
        <form action="" method="post" @change="filterForm">
          <table class="Filter">
            <tr>
              <td>Email менеджера</td>
              <td><input type="text" v-model.trim="emailManager"/></td>
              <td>Email клиента</td>
              <td><input type="text" v-model.trim="emailClient"/></td>
            </tr>
            <tr>
              <td>Обновлено от</td>
              <td>
                <datepicker v-model="filter.dtFrom"></datepicker>
              </td>
              <td>до</td>
              <td>
                <datepicker v-model="filter.dtTo"></datepicker>
              </td>
            </tr>
            <tr>
              <td>Стаж от</td>
              <td>
                <input type="text" v-model.trim="filter.experienceFrom"/>
              </td>
              <td>до</td>
              <td>
                <input type="text" v-model.trim="filter.experienceTo"/>
              </td>
            </tr>
            <tr>
              <td>Возраст от</td>
              <td>
                <input type="text" v-model.trim="filter.ageFrom"/>
              </td>
              <td>до</td>
              <td>
                <input type="text" v-model.trim="filter.ageTo"/>
              </td>
            </tr>
            <tr>
              <td>ЗП от</td>
              <td>
                <input type="text" v-model.trim="filter.priceFrom"/>
              </td>
              <td>до</td>
              <td>
                <input type="text" v-model.trim="filter.priceTo"/>
              </td>
            </tr>
            <tr>
              <td>м / ж (1 или 2)</td>
              <td>
                <input type="text" v-model.trim="filter.sex"/>
              </td>
              <td>Сайт (,)</td>
              <td>
                <input type="text" v-model.trim="filter.site"/>
              </td>
            </tr>
            <tr>
              <td>Список запросов (,)</td>
              <td colspan="3"><textarea style="width: 100%; height:80px;" type="text" v-model.trim="query"/></td>
            </tr>

            <tr>
              <td>
                <Button class="btns" type="button" @click="uploadResume">Загрузить резюме</Button>
              </td>
              <td>
                <Button class="btns" type="button" @click="resetQuery">Очистить запрос</Button>
              </td>
              <td>
                <Button class="btns" type="button" @click="filterForm">Применить фильтр</Button>
              </td>
              <td>
                <Button class="btns" type="button" @click="sendReport">Отправить отчет</Button>
              </td>
            </tr>
          </table>
        </form>
      </div>
      <div>
        <div class="stat">
          <ul>
            <li v-for="item in stat">
              <a href="" v-on:click.prevent="selectQuery(item._id)">{{item._id}} ({{item.count}})</a>
              <a href="" v-if="item._id!=='Все'" v-on:click.prevent="addQuery(item._id)">добавить</a>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="msgs" v-if="showPanel">
      <div class="infoMsg">{{info}}</div>
      <div class="error">{{error}}</div>
      <div class="count-resumes">Найдено резюме: {{searchCount}}</div>
    </div>

    <table class="res">
      <tr v-show="resumes.length" class="tb-head">
        <td>№</td>
        <td>Заголовок</td>
        <td></td>
        <td></td>
        <td>Фото</td>
        <td>Пол</td>
        <td>Телефон</td>
        <td>Email</td>
        <td>Возраст</td>
        <td>Стаж</td>
        <td>ЗП, руб</td>
        <td>Сайт</td>
        <td>Обновление</td>
      </tr>
      <tr v-for="(item,index) in resumes" v-show="item.showF===true">
        <td>{{index+1}}</td>
        <td>
          <div class="title" @click="showDetail(item._id)">{{item.title}}</div>
          <div class="query">{{item.query}}</div>
          <div v-show="item.showD" class="info" v-html="item.info"></div>
        </td>
        <td><a class="print" :href="'/resume/'+item._id+'?utm_campaign='+apiToken" target="_blank">Печать</a></td>
        <td><a class="remove" href="" v-on:click.prevent="remove(item._id)">Удалить</a></td>
        <td>
          <div class="ava"><img v-if="item.imgPublic" :src="'/images/resumes/'+item.imgPublic"></div>
        </td>
        <td>{{item.sex===1?'м':(item.sex===2?'ж':'')}}</td>
        <td>{{item.phone}}</td>
        <td>{{item.email}}</td>
        <td>{{item.age}}</td>
        <td>{{item.experience}}</td>
        <td>{{item.price}}</td>
        <td><a :href="item.url" target="_blank">{{item.site}}</a></td>
        <td>{{new Date(item.updatedAt).toLocaleString().replace(/:00$/,'')}}</td>
      </tr>
    </table>
  </div>
</template>

<script>
  import axios from 'axios'
  import Datepicker from 'vuejs-datepicker'

  export default {
    name: 'app',
    components: {Datepicker},
    data: function () {
      return {
        query: '',
        emailClient: '',
        emailManager: '',
        apiToken: '',
        stat: [],
        info: '',
        error: '',
        resumes: [],
        showPanel: true,
        filter: {
          dtFrom: new Date(Date.now() - (30 * 24 * 60 * 60 * 1000)),
          dtTo: new Date(Date.now() + (24 * 60 * 60 * 1000)),
          ageFrom: null,
          ageTo: null,
          experienceFrom: null,
          experienceTo: null,
          priceFrom: null,
          priceTo: null,
          sex: null,
          site: ''
        }
      }
    },
    computed: {
      searchCount: function () {
        return this.resumes.filter(e => e.showF === true).length || 0
      }
    },
    created: function () {
      this.uploadEmailManager()
      this.uploadStat()
    },
    methods: {
      resetQuery: function () {
        this.resumes = []
        this.query = ''
      },
      selectQuery: function (query) {
        if (query === 'Все') {
          this.query = ''
          for (let i = 0, n = this.stat.length; i < n; i++) {
            if (i !== 0) this.query += this.stat[i]._id + ', '
          }
        } else {
          this.resumes = []
          this.query = query
        }
      },
      addQuery: function (query) {
        this.resumes = []
        this.query = this.query + ', ' + query
      },
      filterForm: function () {
        for (let i = 0, n = this.resumes.length; i < n; i++) {
          if (new Date(this.resumes[i].updatedAt).getTime() < new Date(this.filter.dtFrom).getTime() ||
            new Date(this.resumes[i].updatedAt).getTime() > new Date(this.filter.dtTo).getTime() ||
            (this.filter.experienceFrom && this.resumes[i].experience < parseFloat(this.filter.experienceFrom)) ||
            (this.filter.experienceTo && this.resumes[i].experience > parseFloat(this.filter.experienceTo)) ||
            (this.filter.ageFrom && this.resumes[i].age < parseFloat(this.filter.ageFrom)) ||
            (this.filter.ageTo && this.resumes[i].age > parseFloat(this.filter.ageTo)) ||
            (this.filter.priceFrom && this.resumes[i].price < parseFloat(this.filter.priceFrom)) ||
            (this.filter.priceTo && this.resumes[i].price > parseFloat(this.filter.priceTo)) ||
            (this.filter.sex && this.resumes[i].sex != this.filter.sex) ||
            (this.filter.site && this.filter.site.split(',').indexOf(this.resumes[i].site) === -1)
          ) {
            this.resumes[i].showF = false
          } else {
            this.resumes[i].showF = true
          }
        }
      },
      uploadEmailManager: function () {
        axios.get('/api/email-manager')
          .then(d => {
            this.emailManager = d.data.email
            this.apiToken = d.data.apiToken
          })
          .catch(e => {
            console.log(e)
          })
      },
      uploadStat: function () {
        this.stat = []
        axios.get('/api/resumes-stat')
          .then(d => {
            this.stat = d.data
            let count = 0
            for (let i = 0, n = this.stat.length; i < n; i++) {
              count += this.stat[i].count
            }
            this.stat.splice(0, 0, {_id: 'Все', count: count})
          })
          .catch(e => {
            this.error = 'Данные не могут быть загружены'
            console.log(e)
          })
      },
      uploadResume: function () {
        if (!this.emailManager) {
          this.error = 'Не указан Email менеджера'
          return
        }
        if (!this.query) {
          this.error = 'Пустой запрос'
          return
        }
        this.info = ''
        this.error = ''
        this.resumes = []
        axios.get('/api/resumes?query=' + this.query.toLowerCase())
          .then(d => {
            for (let i = 0, n = d.data.length; i < n; i++) {
              d.data[i].showD = false
              d.data[i].showF = true
            }
            this.resumes = d.data
          })
          .catch(e => {
            this.error = 'Данные не могут быть загружены'
            console.log(e)
          })
      },
      sendReport: function () {
        if (!this.emailManager || !this.emailClient) {
          this.error = 'Не все поля заполнены'
          return
        }
        if (!this.resumes.length) {
          this.error = 'Нет ни одного резюме'
          return
        }
        this.info = ''
        this.error = ''
        let data = {
          emailManager: this.emailManager.toLowerCase(),
          emailClient: this.emailClient.toLowerCase(),
          query: this.query,
          ids: this.resumes.filter(e => e.showF === true).map(e => e._id)
        }
        axios.post('/api/report', data)
          .then(d => {
            this.info = 'Отчет отправлен на ' + this.emailClient
            console.log(d.data)
          })
          .catch(e => {
            this.error = 'Отчет не был отправлен. ' + e.response.data
            console.log(e)
          })
      },
      remove: function (_id) {
        for (let i = 0, n = this.resumes.length; i < n; i++) {
          if (this.resumes[i]._id === _id) {
            this.resumes.splice(i, 1)
            break
          }
        }
      },
      showDetail: function (_id) {
        for (let i = 0, n = this.resumes.length; i < n; i++) {
          if (this.resumes[i]._id === _id) {
            if (this.resumes[i].showD === false) this.resumes[i].showD = true
            else this.resumes[i].showD = false
          } else {
            this.resumes[i].showD = false
          }
        }
      }
    }
  }
</script>
