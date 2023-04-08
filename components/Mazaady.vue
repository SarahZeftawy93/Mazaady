<template>
    <v-container class="d-flex flex-column align-center justify-center">
        <v-row no-gutters>
            <v-col class="custom-title d-flex align-center justify-center">
                <h2 class="text-h2">Mazaady</h2>
            </v-col>
        </v-row>
        <v-row no-gutters>
            <v-col cols="12">
                <v-autocomplete
                    label="Main Category"
                    v-model="mainVal"
                    :items="mazItems"
                    item-text="name"
                    item-value="id"
                    variant="filled"
                    return-object
                    @change="fillSubMaz"
                    required
                >
                </v-autocomplete>
            </v-col>
        </v-row>    
        <v-row no-gutters>
            <v-col cols="12">
                <v-autocomplete
                    label="Sub Category"
                    v-model="subVal"
                    :items="mazSubItems"
                    item-text="name"
                    item-value="id"
                    variant="filled"
                    return-object
                    required
                    @change="getProcessType"
                ></v-autocomplete>
            </v-col>
        </v-row>        
        <v-row v-for="(item, index) in processType">
            <v-col cols="12" v-if="item.options.length > 0">
                <v-autocomplete
                :label="item.name"
                v-model="item.itemValue"
                :items="item.options"
                item-text="name"
                item-value="id"
                variant="filled"
                return-object
                @change="getModels"
                >
                </v-autocomplete>
            </v-col>
            <v-col cols="12" v-if="item.showOther">
                <v-text-field
                    v-model="item.itemOther"
                    label="Other"
                >
                </v-text-field>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <v-btn
                color="success"
                class="mt-4"
                block
                @click="submit"
                >
                Submit
                </v-btn>
            </v-col>
        </v-row>
        <v-row v-if="result.length > 0">
            <div>
                <table>
                    <thead>
                    <tr>
                        <th class="text-left pa-md-4 mx-lg-auto">
                        Key
                        </th>
                        <th class="text-left pa-md-4 mx-lg-auto">
                        Value
                        </th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr
                        v-for="item in result"
                    >
                        <td class="pa-md-4 mx-lg-auto">{{ item.key }}</td>
                        <td class="pa-md-4 mx-lg-auto">{{ (item.value == 'Other'?item.other:item.value) }}</td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </v-row>
    </v-container>
</template>
<script>
import axios from 'axios';
const privateKey = '3%o8i}_;3D4bF]G5@22r2)Et1&mLJ4?$@+16';
export default {
  name: 'Mazaady',
  data() {
        return {
            mainVal:"",
            subVal:"",
            mazItems: [],
            mazSubItems:[],
            processType:[],
            result:[],
        };
    },
    mounted () {
    axios
      .get('https://staging.mazaady.com/api/get_all_cats')
      .then(response => {
        this.mazItems = response.data.data.categories;
    })
  },
  methods:{
    fillSubMaz(value) {
        this.processType = []
        this.mazSubItems = []
        this.result = []
        for(let i = 0; i < this.mazItems.length; i++) {
            if (value) {
                if (this.mazItems[i].id == value.id) {
                    this.mazSubItems = this.mazItems[i].children;
                }
            }
        }
    },
    getProcessType(value) {
        if (value) {
            axios
            .get('https://staging.mazaady.com/api/v1/properties?cat='+value.id,
            { headers: {"private-key" : `${privateKey}`}})
            .then(response => {
                this.processType = response.data.data;
                for (let i = 0; i < this.processType.length; i++) {
                    this.processType[i].options.push({
                        child:false,
                        id:"other",
                        name:"Other",
                        parent:this.processType[i].id,
                        slug:"other",
                        showOther:true
                    })
                }
            })
        }
    },
    getModels(value) {
        for (let i = 0; i < this.processType.length; i++) {
            if (this.processType[i].id == value.parent) {
                if (value.id == 'other') {
                    this.processType[i].showOther = true
                } else {
                    this.processType[i].showOther = false
                }
            }
        }
        if (value.parent == 2) {
            axios
            .get('https://staging.mazaady.com/api/v1/get-options-child/'+value.id,
            { headers: {"private-key" : `${privateKey}`}})
            .then(response => {
                response.data.data[0].options.push({
                    child:false,
                        id:"other",
                        name:"Other",
                        parent:2,
                        slug:"other",
                        showOther:true
                })
                this.processType.splice(2, 0, response.data.data[0])
            })
        } else if (value.parent == 647) {
            axios
            .get('https://staging.mazaady.com/api/v1/get-options-child/'+value.id,
            { headers: {"private-key" : `${privateKey}`}})
            .then(response => {
                response.data.data[0].options.push({
                    child:false,
                        id:"other",
                        name:"Other",
                        parent:2,
                        slug:"other",
                        showOther:true
                })
                this.processType.splice(3, 0, response.data.data[0])
            })
        }
    },
    submit() {
        this.result = [{
                "key":"Main Category",
                "value":this.mainVal.name,
                "other":""
            },
            {
                "key":"Sub Category",
                "value":this.subVal.name,
                "other":""
            },
            
        ]
        for (let i = 0; i < this.processType.length; i++) {
            console.log(this.processType);
            this.result.push({
                "key":this.processType[i].name,
                "value":(this.processType[i].itemValue?this.processType[i].itemValue.name:""),
                "other":(this.processType[i].itemOther || '')
            })
        }
    }
  }
}
</script>