<template>
	<v-data-table
    :headers="headers"
    :items="doors"
    :loading="load"
    sort-by="id"
    single-line
    class="elevation-1"
  >
      <!-- @click:row="handleClick" -->
    <template v-slot:item.id="{ item }">
      <v-chip
        :color="getColor(item.id)"
        dark
      >
        {{ item.id }}
      </v-chip>
    </template>
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Заказы</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-toolbar-title>Сегодня: 14.10.2020</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              Добавить
            </v-btn>
          </template>
       

          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.name"
                      label="Dessert name"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.calories"
                      label="Calories"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.fat"
                      label="Fat (g)"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.carbs"
                      label="Carbs (g)"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.protein"
                      label="Protein (g)"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>


	<script>
  import axios from 'axios'
  export default {
    data: () => ({
      dialog: false,
      dialogDelete: false,
      load: true,
      headers: [
  
         { text: 'Ред.', value: 'actions', sortable: false },
        { text: 'Код', value: 'id' },
        { text: 'Время монтажа', value: 'time_mont' },
        { text: 'БР', value: 'brigada_mont.name' },
        { text: 'Часть города', value: 'part_city' },
        { text: 'Адрес', value: 'adres' },
        {text: 'Продавец', sortable: true, value: 'saler'},
        {text: 'Статус заявки', value: 'status'},
        {text: 'Дата', value: 'date'},
        {text: 'Телефон', value: 'phone'},
        {text: 'ФИО', value: 'fio'},
        {text: 'Модель двери продавца', value: 'model_saler.name'},
        {text: 'Модель двери рук', value: 'id'},
        {text: 'Размер двери', value: 'door_size'},
        {text: 'Сторона откр', value: 'door_direction'},
        {text: 'Размер проема', value: 'proem_size'},
        {text: 'Желаемая дата монтажа', value: 'date_mont'},
        {text: 'Замерщик', value: 'zamershik.name'},
        {text: 'Статус бухгалтера', value: 'status_premia'},
        {text: 'Цена диллера', value: 'cost_diler'},
        {text: 'Сумма премии', value: 'sum_premia'},
        {text: 'Премия ВДЗ', value: 'vdz_premia'},
        {text: 'Примечание продавца', value: 'prim_saler'},
        {text: 'Примечание Руководителя', value: 'prim_rukvod'},
      ],
      doors: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      defaultItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      


      // api 

        this.load = true
      axios
      .get('https://door.webink.site/wp-json/door/v1/get/sales')
      .then(response => {
        console.log(response.data)
       this.doors = response.data

       this.load = false
      })


    },

    methods: {
    	// handleClick(value){
    	// 	console.log(value.calories)
    	// 	this.$router.push('/' + value.calories)
    	// },
    getColor (calories) {
        if (calories > 400) return 'red'
        else if (calories > 200) return 'orange'
        else return 'green'
      },
      editItem (item) {
        this.editedIndex = this.doors.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.doors.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteItemConfirm () {
        this.doors.splice(this.editedIndex, 1)
        this.closeDelete()
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save () {
        if (this.editedIndex > -1) {
          Object.assign(this.doors[this.editedIndex], this.editedItem)
        } else {
          this.doors.push(this.editedItem)
        }
        this.close()
      },


    },
  }
</script>