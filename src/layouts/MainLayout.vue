<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          Simple Sales System
        </q-toolbar-title>

        <div>Filipe Eduardo Regis</div>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <div id="container">
        <div id="contentContainer">
          <div id="doubleInputContainer">
            <q-input class="inputFormat" outlined v-model="inputs.order_number.text" label="* Número do Pedido" :dense="false" />
              <q-input class="inputFormat" outlined v-model="inputs.emission_date.date" label="* Data Emissão" mask="##/##/####">
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                      <q-date v-model="inputs.emission_date.date" mask="DD-MM-YYYY">
                        <div class="row items-center justify-end">
                          <q-btn v-close-popup label="Close" color="primary" flat />
                        </div>
                      </q-date>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
          </div>
          <q-select class="inputFormat" outlined v-model="inputs.customer.text" :options="inputs.options" label="* Cliente" :dense="false" />

          <div class="buttonsContainer">
            <q-btn class="buttonFormat" @click="addOrUpdateSet('add')" color="primary" label="Adicionar Produto" />
            <q-btn class="buttonFormat" @click="addOrUpdateSet('update')" :disabled="selected.length === 0" color="primary" label="Editar" />
            <q-btn class="buttonFormat" @click="confirm = true" :disabled="selected.length === 0" color="primary" label="Excluir" />
          </div>

          <div class="q-pa-md table">
            <q-table
              title="Vendas"
              :data="data"
              :columns="columns"
              row-key="id"
              selection="single"
              :selected.sync="selected"
            />

            <div id="tableInfoContainer" class="q-mt-md">
              <div id="tableInfoContent">
                <p> Total Pedido: R${{total}} </p>
                <p> Quantidade Itens: {{data.length}} </p>
              </div>
            </div>
          </div>
          <div class="buttonsContainer">
            <q-btn class="buttonFormat" @click="validarCampos()" :disabled="isDisabledButtonsFooter"  color="primary" label="Salvar" />
            <q-btn class="buttonFormat" @click="validarCamposSalvar()" :disabled="isDisabledButtonsFooter"  color="primary" label="Salvar/Novo" />
          </div>
        </div>
        <q-dialog v-model="confirm" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <span class="q-ml-sm">Tem certeza que deseja excluir?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Não" color="primary" v-close-popup />
          <q-btn flat label="Sim, to seguro" color="primary" @click="removeProduct()" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
        <q-dialog v-model="prompt" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div v-if="isAdd" class="text-h6">Cadastro de Produto</div>
          <div v-else class="text-h6">Alteração de Produto</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input class="inputFormat" outlined v-model="inputs.product_name.text" label="* Produto" :dense="false" />
          <q-field class="inputFormat" outlined  label="Quantidade" :dense="false">
            <template v-slot:control>
              <q-slider class="inputFormat" outlined v-model="inputs.amount.text" :min="1" :max="20" :step="1" label :dense="false" />
            </template>
          </q-field>
          <q-input class="inputFormat" mask="#.##" fill-mask="0" reverse-fill-mask outlined v-model="inputs.product_price.text" label="* Valor Unitário" :dense="false" />
         <q-field class="inputFormat" outlined  label="Valor Desconto" :dense="false">
            <template v-slot:control>
              <q-slider class="inputFormat" outlined v-model="inputs.discount_price.text" :min="0" :max="inputs.product_price.text" :step="1" label :dense="false" />
            </template>
          </q-field>
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancel" @click="onClose()" v-close-popup />
          <q-btn flat v-if="isAdd" label="Adicionar" @click="addProduct()" :disabled="isDisabled" v-close-popup/>
          <q-btn flat v-else label="Atualizar" @click="updateProduct()" :disabled="isDisabled" v-close-popup/>
        </q-card-actions>
      </q-card>
    </q-dialog>
      </div>
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  name: 'MainLayout',
  components: { },
  data () {
    return {
      codigo: 1,
      prompt: false,
      confirm: false,
      isAdd: false,
      inputs: {
        order_number: {
          text: ''
        },
        emission_date: {
          date: ''
        },
        customer: {
          text: ''
        },
        product_name: {
          text: ''
        },
        amount: {
          text: 1
        },
        product_price: {
          text: ''
        },
        discount_price: {
          text: 0
        },
        options: [
          'StudioZ', 'Andreiffs', 'BGO', 'C&A', 'Beagle'
        ]
      },
      selected: [],
      columns: [
        {
          name: 'id',
          required: true,
          label: 'Código',
          align: 'left',
          field: row => row.id,
          sortable: true
        },
        { name: 'products', align: 'center', label: 'Produto', field: 'products', sortable: true },
        { name: 'amount', align: 'center', label: 'Quantidade', field: 'amount', sortable: true },
        { name: 'product_price', label: 'Valor Unitário', field: 'product_price', sortable: true, format: val => `R$${val}` },
        { name: 'discount_price', label: 'Valor Desconto', field: 'discount_price', format: val => `R$${val}` },
        { name: 'total', label: 'Total', field: 'total', format: val => `R$${val}` }
      ],
      data: []
    }
  },
  methods:
  {
    testes: function () {
      alert('oi')
    },
    addProduct: function () {
      this.data.push({
        id: this.codigo++,
        products: this.inputs.product_name.text,
        amount: this.inputs.amount.text,
        product_price: this.inputs.product_price.text,
        discount_price: this.inputs.discount_price.text,
        total: (this.inputs.amount.text * this.inputs.product_price.text) - this.inputs.discount_price.text
      })
      this.onClose()
    },
    addOrUpdateSet: function (transactionType) {
      if (transactionType === 'add') {
        this.isAdd = true
        this.prompt = true
      } else {
        this.inputs.product_name.text = this.selected[0].products
        this.inputs.amount.text = this.selected[0].amount
        this.inputs.product_price.text = this.selected[0].product_price
        this.inputs.discount_price.text = this.selected[0].discount_price
        this.prompt = true
      }
    },
    removeProduct: function () {
      var removeIndex = this.data.map(function (item) { return item.id }).indexOf(this.selected[0].id)

      this.data.splice(removeIndex, 1)

      this.selected = []
    },
    updateProduct: function () {
      var updateIndex = this.data.map(function (item) { return item.id }).indexOf(this.selected[0].id)
      console.log(updateIndex)
      console.log(this.data)

      this.data[updateIndex].products = this.inputs.product_name.text
      this.data[updateIndex].amount = this.inputs.amount.text
      this.data[updateIndex].product_price = this.inputs.product_price.text
      this.data[updateIndex].discount_price = this.inputs.discount_price.text
      this.data[updateIndex].total = (this.inputs.amount.text * this.inputs.product_price.text) - this.inputs.discount_price.text

      this.selected = []
      this.onClose()
    },
    onClose: function () {
      this.isAdd = false

      this.inputs.product_name.text = ''
      this.inputs.amount.text = 1
      this.inputs.product_price.text = 0
      this.inputs.discount_price.text = 0
      this.prompt = true
    },
    validarCampos: function () {
      if (this.customer !== '' && this.order_number !== '' && this.emission_date !== '') {
        alert('Validado com sucesso')
        this.inputs.customer.text = ''
        this.inputs.order_number.text = ''
        this.inputs.emission_date.date = ''
        this.data = []
        this.selected = []
      }
    },
    validarCamposSalvar: function () {
      if (this.customer !== '' && this.order_number !== '' && this.emission_date !== '') {
        alert('Validado e salvo com sucesso')
        this.inputs.customer.text = ''
        this.inputs.order_number.text = ''
        this.inputs.emission_date.date = ''
        this.data = []
        this.selected = []
      }
    }
  },
  computed: {
    isDisabled: function () {
      return this.inputs.product_name.text.length < 1 || this.inputs.product_price.text.length < 1
    },
    isDisabledButtonsFooter: function () {
      return this.data.length < 1 || this.inputs.emission_date.date.length < 1 || this.inputs.order_number.text === '' || this.inputs.customer.text === ''
    },
    // eslint-disable-next-line eqeqeq
    total: function () { return this.data.length === 0 ? 0 : this.data.reduce((a, b) => ({ total: a.total + b.total })).total }
  }
}

</script>

<style>
  #container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    width: 100%;
    height: 90vh;
  }

  #contentContainer {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    max-width: 980px;
    width: 100%;
  }

  #doubleInputContainer {
    display: flex;
    justify-content: center;
    align-items: center;

    width: 100%;
  }

  .buttonsContainer {
    display: flex;
    justify-content: flex-start;
    align-items: center;

    width: 100%;
  }

  .buttonFormat {
    padding: 4px;
    margin: 10px;
  }

  .table {
    width: 100%;
  }

  #tableInfoContainer {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    flex-direction: row;

    width: 100%;
  }

  #tableInfoContent {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    font-size: 18px;
  }

  .inputFormat {
    padding: 4px;
    width: 100%;
  }
</style>
