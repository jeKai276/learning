<template>
    <v-card
      class="mb-5"
      height="600px">
      <v-card-text>
        <v-form ref="form" class="px-3">
            <div>
                <div v-if="alt" class="alert alert-error" style="    margin-left: 11%;
    width: 74%;">
                    <i class="uiIconClose" @click="DontShow"></i>Remplire tout les champs
                </div>
            </div>
          <v-container>
            <v-flex md10>
              <v-text-field
                v-model="nameCourse"
                label="Libellé"
                prepend-icon="folder"
                :rules="inputRules" />
            </v-flex>
            <v-layout>
              <v-flex md6>
                <p class=" text-md-left subheading  font-weight-light blue-grey--text text--darken-1">Catégorie de cours</p>
              </v-flex>
            </v-layout>

            <v-layout>
              <v-flex md8>
                <select
                  v-model="selectedCategory"
                  class="select_style">
                  <option value disabled>select category</option>
                  <option v-for="option in category"  :value="option.idCategory">
                    {{ option.nameCategory }}
                  </option>
                </select>
              </v-flex>
            </v-layout>
              <!-- pop up -->
              <v-dialog v-model="dialogs" width="600px">
                  <v-btn
                          slot="activator"
                          fab
                          dark
                          color="#578dc9"
                          small>
                      <v-icon dark>add</v-icon>
                  </v-btn>
                  <v-card>
                      <v-card-title>
                          <h4>Add New Category</h4>
                      </v-card-title>
                      <v-card-text>
                          <v-form ref="form" class="px-3">
                                  <v-text-field
                                          v-model="nameCategory"
                                          label="libellé Categorie"
                                          prepend-icon="folder"
                                          :rules="inputRules" />
                                  <v-spacer />
                                  <button
                                          type="button"
                                          class="btn btn-primary"
                                  @click="saveCategory">
                                      Ajouter
                                  </button>
                                  <button
                                          class="btn"
                                  @click="cancel">
                                      Quitter
                                  </button>
                          </v-form>
                      </v-card-text>
                  </v-card>
              </v-dialog>
             <!-- end popup -->

            <v-layout>
              <v-radio-group v-model="courseStatus" row>
                <v-radio
                  label="Draft"
                  color="blue"
                  value="DRAFT" />
                <v-radio
                  label="Archived"
                  color="blue"
                  value="ARCHIVED" />
                <v-radio
                  label="Published"
                  color="blue"
                  value="PUBLISHED" />
              </v-radio-group>
            </v-layout>
            <v-flex md10>
              <v-text-field
                      type="number"
                v-model="nbPerson"
                label="Nbre des personnes"
                prepend-icon="folder" />
            </v-flex>
            <v-layout>
              <v-flex md6>
                <p class=" text-md-left subheading  font-weight-light blue-grey--text text--darken-1">Date de début</p>
                <Datepicker
                  id="dateStart"
                  v-model="dateStart"
                  placeholder="Select Date Start" 
                  monday-first="true"
                  :format="formatDate"
                  :language="fr" />
              </v-flex>
              <v-flex md6>
                <p class=" text-md-left subheading  font-weight-light blue-grey--text text--darken-1">Date de Fin</p>
                <Datepicker
                  id="dateEnd"
                  v-model="dateEnd"
                  placeholder="Select Date End" 
                  monday-first="true"
                  :format="formatDate"
                  :language="fr" />
              </v-flex>
              <v-spacer />
            </v-layout>
            <v-layout>
              <v-flex md4>
                <p class=" text-sm-left subheading  font-weight-light blue-grey--text text--darken-1">Récompense</p>
              </v-flex>
              <v-layout>
                <v-flex md4>
                  <v-checkbox
                    v-model="rewardCourse"
                    label="Badge"
                    value="badge" />
                </v-flex>
                <v-flex>
                  <v-checkbox
                    v-model="rewardCourse"
                    label="Certification"
                    value="certification" />
                </v-flex>
              </v-layout>
            </v-layout>
            <v-layout>

            </v-layout>
            <v-spacer />

              <button
                type="button" class="btn btn-primary"
                @click.prevent="onSubmit">
                Ajouter
              </button>
          </v-container>
        </v-form>
      </v-card-text>
    </v-card>
</template>
<script>
    import axios from 'axios'
    import { bus } from '../../main';
    import Datepicker from 'vuejs-datepicker';
    import {en, fr} from 'vuejs-datepicker/dist/locale'
    import moment from 'moment';


    Vue.filter('formatDate', function(value) {
        if (value) {
            const lang = eXo.env.portal.language;
            moment.locale(lang);
            return moment(String(value), value.includes('/') ? 'DD/MM/YYYY' : 'YYYY-MM-DD').format(lang == 'fr' ? 'DD MMMM YYYY' : 'MMMM DD YYYY');
        }
    });

    export default {
        props: ['contentcourse'],
        components: {
            Datepicker
        },
        props: {
            course: {
                type: Object,
                default: () => null
            }
        },
        data: function () {
            return {
                alt: false,
                category: {
                    nameCategory: ''
                },
                dialogs: false,
                selectedCategory: '',
                categories: {
                    idCategory:null,
                    nameCategory: ''
                },
                en: en,
                fr: fr,
                date: new Date(),
                config: {
                    format: 'YYYY-MM-DD',
                    useCurrent: false,
                },
                inputRules: [
                    v => !!v || 'This field is required',
                    v => v.length >= 3 || 'Minimum length is 3 characters'
                ],
                nameCategory: '',
                e1: 0,
                nameCourse: '',
                courseStatus: 'DRAFT',
                rewardCourse: '',
                dateStart: '',
                dateEnd: '',
                nbPerson: '',
                courses: {
                    nameCourse: '',
                    dateStart: '',
                    dateEnd: '',
                    nbPerson: '',
                    rewardCourse: '',
                    idCategory: '',
                    status: '',
                    icon: null,
                    iconFileId: null,


                },
                addSuccess: false,
                addError: false,
                updateMessage: ''
            }
        },
        mounted() {
            axios.get(`/portal/rest/category/all`)
                .then(response => {
                    this.category = response.data

                })
                .catch(e => {
                    this.errors.push(e)
                })
        },
        methods: {
            DontShow() {
                this.alt = false;
            },
            quitter: function () {
                this.$router.push('/createCours')
            },
            onSubmit() {
                if(this.nameCourse===''|| this.dateStart===''||this.dateEnd===''||this.nbPerson===null||this.selectedCategory===''){
                    this.alt=true;
                }
                if (this.alt === false) {
                    this.courses.nameCourse = this.nameCourse;
                this.courses.dateStart = this.dateStart;
                this.courses.dateEnd = this.dateEnd;
                this.courses.nbPerson = this.nbPerson;
                this.courses.rewardCourse = this.rewardCourse;
                this.courses.status = this.courseStatus;
                this.courses.idCategory = this.selectedCategory;
                    axios.post(`/portal/rest/cours/add`, this.courses, {
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    }).then((response) => {
                        this.courses = response.data
                        bus.$emit('CourseChanged', this.courses);
                        console.log("course changed", this.courses)
                    })
                }
            },
            saveCategory() {
                this.categories.nameCategory = this.nameCategory;
                axios.post('/portal/rest/category/add', this.categories, {
                    headers: {
                        'Content-Type': 'application/json'
                    }

                }).then((response) => {
                    console.log(response.data)
                    this.category.push(response.data)
                });
                this.dialogs = false

            },
            cancel() {
                this.dialogs = false;
            }
        }
    }
</script>
<style>
    .v-dialog__activator {
        margin-left: 417%;
        margin-top: -130%;
    }
</style>

