<template>
    <div class="bg-indigo-200">
    {{carregarValores()}}
        <div class="grid grid-cols-1 lg:grid-cols-2">
        <div class="bg-blue-600  lg:h-screen lg:justify-center lg:min-h-screen lg:flex lg:items-center p-8 sm:p-12">
            <div class="flex-grow">
                <h1 class="text-white text-center text-2xl sm:text-5xl mb-2 self-auto">Cryptomoeda</h1><br>
                <p class="text-center text-lg text-blue-200">Pesquise uma Cryptomoeda pelo seu ticker!</p><br>
                <div class="items-center flex ">
                    <input class="flex-1 w-full mr-2 text-gray-700 px-4 py-1 bg-gray-300 rounded-md hover:bg-gray-200 focus:bg-white focus:outline-none border border-gray-300" type="text" placeholder="Ticker da cryptomoeda! Ex: USDT/BTC" name="author" v-model="name">
                    <button class="flex-shrink-0 text-white hover:text-gray-700 border border-white hover:bg-white font-semibold rounded-md text-lg px-4 py-1 focus:outline-none" type="submit" v-on:click="adicionarMoeda" >Adicionar</button>
                </div>
            </div>
        </div>
        <div class="sm:min-h-screen flex items-center p-3 lg:p-24 xl:p-48 h-screen justify-center items-center">
            <div class="overscroll-auto overflow-y-scroll p-1  h-full shadow-xl ">
                <div class="bg-white rounded-md border border-gray-300 p-2" v-for="(comment, index) in listaDeMoedas" v-bind:key="index">
                    <div class="">
                        <div class="items-center flex text-center flex justify-content align-center">
                            <div class="flex-none text-xl">{{comment.name}}</div>
                            <div class="text-sm text-gray-600 w-full flex-1 px-1">Cotação ${{comment.valor}}</div>
                            <button class="flex-none inset-y-0 right-0 text-red-500 hover:text-white border border-red-500 hover:bg-red-500 font-semibold rounded-md text-sm px-4 py-1 focus:outline-none" v-on:click="removerMoeda(index)">Apagar</button>
                        </div>
                    </div>
                    <br>
                </div>
            </div>
        </div>
    </div>
    </div> 
</template>

<script>
let apikey = "71C52D18-5023-4D19-AB85-7287D7E54D02"
export default{
   
    name: 'moedas',
    data() {
        return {
                    moeda: [{ "name": "BTC", "valor": "" }, { "name": "ETH", "valor": "" }, { "name": "BNB", "valor": "" }, { "name": "SHIB", "valor": "" },{ "name": "BCOIN", "valor": "" }],
                    name: '',
                    valor: "",
                    siglas: "",
                    nome_moeda: "",
                    verificar_erro : 0,
                    cadeira: "",
                    backup : 0,
                    verificacao_lista:0
                }
    
}, methods: {
    mounted(element,operador) {
        if (operador == 1){
            this.$axios.$get("https://rest.coinapi.io/v1/assets?filter_asset_id="+ element , { headers: {"X-CoinAPI-Key": apikey}})
            .then (response =>{
                this.cadeira = [response];
                this.atribuirValores()})
                } else{
                    this.$axios.$get("https://rest.coinapi.io/v1/exchangerate/"+ element +"/USD" , { headers: {"X-CoinAPI-Key": apikey}})
                    .then (response =>{
                        this.cadeira = response,
                        this.moeda.forEach(mylist => {
                            if (mylist.__ob__.value.name === this.cadeira.__ob__.value.asset_id_base){
                                mylist.__ob__.value.valor = this.cadeira.__ob__.value.rate}
                                });
                            this.atualizarMoedas()
                                })
                    .catch(error =>{ this.verificar_erro= error
                    
                    this.moeda.splice(-1,1),
                    alert("Esta cryptomoeda não foi encontrada!")}
                            )
                   }
                },
    atribuirValores(){
        this.cadeira[0].forEach(api => {
            this.moeda.forEach(mylist => {
                if (mylist.__ob__.value.name === api.asset_id){
                    this.backup =  mylist.__ob__.value.valor
                    mylist.__ob__.value.valor = api.price_usd
                    if ( mylist.__ob__.value.name === 'USD'){
                        //console.log("to caindo na condição")
                        mylist.__ob__.value.valor = this.backup
                        this.backup = 0
                        }
                         }
                            });
                        });

                },
                atualizarMoedas(){
                    this.nome_moeda = ''
                    this.moeda.forEach(element => {
                    this.nome_moeda = this.nome_moeda + element.name + ","});
                    this.nome_moeda = this.nome_moeda.substring(0, this.nome_moeda.length - 1)
                    this.mounted(this.nome_moeda,1)
                },
     adicionarMoeda(){
        if ( this.name.trim()===''){
            return;
        }
        this.name = this.name.toUpperCase()
        if (this.verificacao_lista === 0){
            this.moeda.forEach(element => {
                if (this.name===element.name){
                    this.verificacao_lista = 1
                }
            });
        }
        if (this.verificacao_lista === 1){
            alert("Esta cryptomoeda já está na lista!")
            this.verificacao_lista = 0,
            this.name = ""
            return
        } else {
            this.moeda.push({
            name: this.name,
            valor: this.valor
            });
            this.mounted(this.name,0)
            this.name ="",
            this.valor= ""
                    }
                    
                        
                   
     },removerMoeda(index){
        this.moeda.splice(index,1);
     },
     carregarValores(){
                    if (this.verificar_erro === 0){
                        this.verificar_erro = this.verificar_erro + 1,
                        this.atualizarMoedas()
                    }else{
                        setTimeout(() => { this.atualizarMoedas(),
                        console.log("bateu 30") } , 30000)
                    }
                }
     
     
    },
    computed: {
        listaDeMoedas(){
 return this.moeda.map(comment => ({
                        ...comment
                        
                    }))
        }
    }

}
</script>