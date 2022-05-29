<template>
  <div class="PhotoItem">
    <img :src="photo.url" @click="showModal(photo)" alt=""/>
  </div>
   <!--Модальное окно - фото-->
    <div v-if="modalShow" class="modal-shadow" @click.self="closeModal">
      <div class="modal">
        <div class="modal-close" @click="closeModal">&#10006;</div>
          <slot name="title">
            <h3 class="modal-title">Фото</h3>
          </slot>
          <slot name="body">
            <div class="modal-content">
              <img :src='selectedImgPath' alt=""/>
            </div>
          </slot>
          <slot name="footer">
            <div class="modal-footer">
              <LoaderSpin v-if="loading"/>
              <ul v-else-if="allComments.length" class="Comment-list">
                <li>Комментарии</li>
                <li class="Comment-text" v-for="comment in allComments"
                     v-bind:key="comment.id">
                     {{comment.text}}
                </li>
              </ul>
              <p v-else>Комментарии не найдены:(</p>
              <button class="modal-footer__button" @click.self="OpenModalComments">
                Добавить комментарий
              </button>
            </div>
          </slot>
      </div>
    </div>

  <!--Модальное окно для комментариев-->
  <div v-if="modalCommentsShow" class="modal-shadow" @click.self="closeModalComments">
    <div class="modal">
      <div class="modal-close" @click="closeModalComments">&#10006;</div>
      <slot name="title">
        <h3 class="modal-title">Добавить комментарий</h3>
      </slot>
      <slot name="body">
        <div class="modal-content">
          <form v-on:submit.prevent="submitForm">

            <div class="form-group">
              <label for="name">Введите имя: </label>
              <input type="text" class="form-control" id="name" placeholder="Ваше имя" v-model="form.name">
            </div>
            <label for="text">Ваш комментарий</label>
            <div class="form-group">

              <textarea name="text" class="form-control" id="text" v-model="form.comment"></textarea>
            </div>

            <div class="form-group">
              <button class="modal-footer__button">Оставить комментарий</button>
            </div>
          </form>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
import LoaderSpin from '@/components/LoaderSpin'
import axios from 'axios'
export default {
  props: {
    photo: {
      type: Object,
      required: true
    },
  },
  components:{
    LoaderSpin
  },
  data() {
    return {
      modalShow: false,
      modalCommentsShow: false,
      selectedPhoto:Object, //выбранное фото
      selectedImgPath: String,
      selectImgId: null,
      allComments: [],
      loading: true, //загрузка спиннер
      form: {
        id: null,
        name: '',
        photoId:null,
        comment: '',
        text:'',
        date:''
      }
    };
  },
  methods: {
    showModal(Photo) {
      this.selectedPhoto = Photo
      this.selectedImgPath = Photo.url
      this.selectImgId = Photo.id
      this.modalShow = true
      this.loading =true;
      axios
          .get('https://boiling-refuge-66454.herokuapp.com/images/'+this.selectImgId)
          .then(response => (this.allComments = response.data.comments))

      //добавление к найденным комментариям наши собственные
      setTimeout(()=>{if (this.form.photoId == this.selectImgId) this.allComments.push(this.form); this.loading =false;},500);

      },

    OpenModalComments() {
      this.modalCommentsShow = true
      this.modalShow = false
    },
    submitForm(){
      this.form.id = this.allComments.length +1;
      this.form.photoId =  this.selectImgId;
      this.form.text = this.form.comment;
      this.form.date = Date.now();
      axios
          .post('https://boiling-refuge-66454.herokuapp.com/images/'+this.selectImgId+'/comments',this.form)
          .then((res) => {
            if (res.status === 204) {
              //проверка на отправленность комментария пройдена
              alert('Комментарий успешно добавлен!');
              //закрываем окно
              this.modalCommentsShow = false;
              this.showModal(this.selectedPhoto);
              //возвращаем промис
              return new Promise((resolve) => resolve(null))
            }
            if (!res.ok) {
              alert('Комментарий не сохранился:(');
              throw new Error(res.statusText)
            }
            return res.json();
          })
          .catch((error) => {
              alert('Заполните все поля!');
              console.log(error)
          })
    },
    closeModal: function () {
      this.modalShow = false
    },
    closeModalComments: function (){
      this.modalCommentsShow = false
    }
  },
}
</script>

<style scoped lang="scss">

.PhotoItem{
  margin: 8px;
  cursor: pointer;
  transition: 1s;
}
.PhotoItem:hover{
  transform: scale(1.05);
}

//модальное окно
.modal-shadow {
  position: absolute;
  top: 0;
  left: 0;
  min-height: 100%;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.39);
}
.modal {
  background: #fff;
  border-radius: 8px;
  padding: 15px;
  min-width: 420px;
  max-width: 550px;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  &-close {
    border-radius: 50%;
    color: #fff;
    background: #2a4cc7;
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    top: 7px;
    right: 7px;
    width: 30px;
    height: 30px;
    cursor: pointer;
  }
  &-title {
    color: #0971c7;
  }
  &-content {
    margin-bottom: 20px;
  }
  &-footer {
    &__button {
      background-color: #0971c7;
      color: #fff;
      border: none;
      text-align: center;
      padding: 8px;
      font-size: 17px;
      font-weight: 500;
      border-radius: 8px;
      min-width: 150px;
      cursor: pointer;
    }
  }
}
//увеличенное фото
.modal-content img{
  width: 450px;
}

//форма заполнения
.form-group{
  margin-bottom: 10px;
}
textarea{
  width: 80%;
  height: 200px;
}

//стили для списка комментариев
.Comment-list {
  list-style: none;
  padding: 0;
  border: 1px solid rgba(0,0,0, .2);
}
.Comment-list li {
  overflow: hidden;
  padding: 6px 10px;
  font-size: 18px;
}
.Comment-list li:first-child {
  font-weight: bold;
  padding: 10px 0 10px 10px;
  margin-bottom: 10px;
  border-bottom: 1px solid rgba(0,0,0, .2);
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
  color: #679bb7;
  font-size: 24px;
  box-shadow: 0 10px 20px -5px rgba(0,0,0, .2);
  text-align: start;
}
.Comment-list li:first-child:before {
  content: "\270E";
  margin-right: 10px;
}

</style>