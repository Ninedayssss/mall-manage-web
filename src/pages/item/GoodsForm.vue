<template>
  <v-form v-model="valid" ref="myGoodForm">
   <v-layout row>
              <v-flex xs5>
                <!--商品分类-->
                <v-cascader
                  url="/item/category/list"
                  required
                  showAllLevels
                  v-model="goods.categories"
                  label="请选择商品分类"/>
              </v-flex>
              <v-spacer/>
              <v-flex xs5>
                <!--品牌-->
                <v-select
                  :items="brandOptions"
                  item-text="name"
                  item-value="id"
                  label="所属品牌"
                  v-model="goods.brandId"
                  required
                  autocomplete
                  clearable
                  dense chips
                  :rules="[v => !!v || '品牌不能为空']"
                >
                  <template slot="selection" slot-scope="data">
                    <v-chip small>{{ data.item.name}}</v-chip>
                  </template>
                </v-select>
              </v-flex>
            </v-layout>
            <v-text-field label="商品标题" v-model="goods.title" :counter="200" required :rules="[v => !!v || '商品标题不能为空']" hide-details/>
              <v-text-field label="商品价格" v-model="goods.price" :counter="200" hide-details/>
            <!-- <v-text-field label="商品卖点" v-model="goods.subTitle" :counter="200" hide-details/> -->
            
      <v-btn @click="submit" color="primary">提交</v-btn>
      <!-- <v-btn @click="clear">重置</v-btn> -->
    </v-layout>
  </v-form>
</template>

<script>
export default {
  name: "goods-form",
  props: {
    oldGoods: {
      type: Object
    },
    isEdit: {
      type: Boolean,
      default: false
    },
    step: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      valid:false,
      goods: {
        categories: [], // 商品分类信息
        brandId: 0, // 品牌id信息
        title: "", // 标题
        //subTitle: "", // 子标题
        price: ""
        // spuDetail: {
        //   packingList: "", // 包装列表
        //   afterService: "", // 售后服务
        //   description: "" // 商品描述
        // }
      },
      brandOptions: [], // 品牌列表
      //specs: [], // 规格参数的模板
      //specialSpecs: [] // 特有规格参数模板
    };
  },
  methods: {
    submit() {
        // 表单校验
        //if (this.$refs.basic.validate()) {
          // 先处理goods，用结构表达式接收,除了categories外，都接收到goodsParams中
      const {
        categories: [{ id: cid1 }, { id: cid2 }, { id: cid3 }],
        ...goodsParams
      } = this.goods;
      goodsParams.cid1 = cid1
      goodsParams.cid2 = cid2
      goodsParams.cid3 = cid3
          //let newObj ={}
          //Object.assign(newObj,goodsParams,obj)
          console.log(goodsParams)
          // 将数据提交到后台
          // this.$http.post('/item/brand', this.$qs.stringify(params))
          this.$http({
            method: this.isEdit ? 'put' : 'post',
            url: '/item/goods',
            data: goodsParams
          }).then(() => {
            // 关闭窗口
            this.$emit("close");
            this.$message.success("保存成功！");
          })
            .catch(() => {
              this.$message.error("保存失败！");
            });
        }
      //},
     
    },
  watch: {
    oldGoods: {
      deep: true,
      handler(val) {
        if (!this.isEdit) {
          Object.assign(this.goods, {
            categories: null, // 商品分类信息
            brandId: 0, // 品牌id信息
            title: "", // 标题
            //subTitle: "", // 子标题
            price: ""
            // spuDetail: {
            //   packingList: "", // 包装列表
            //   afterService: "", // 售后服务
            //   description: "" // 商品描述
            // }
          });
          // this.specs = [];
          // this.specialSpecs = [];
        } else {
          this.goods = Object.deepCopy(val);

          // 先得到分类名称
          const names = val.cname.split("/");
          // 组织商品分类数据
          this.goods.categories = [
            { id: val.cid1, name: names[0] },
            { id: val.cid2, name: names[1] },
            { id: val.cid3, name: names[2] }
          ];

          // 将skus处理成map
          // const skuMap = new Map();
          // this.goods.skus.forEach(s => {
          //   skuMap.set(s.indexes, s);
          // });
          // this.goods.skus = skuMap;
        }
      }
    },
    "goods.categories": {
      deep: true,
      handler(val) {
        // 判断商品分类是否存在，存在才查询
        if (val && val.length > 0) {
          // 根据分类查询品牌
          this.$http
            .get("/item/brand/cid/" + this.goods.categories[2].id)
            .then(({ data }) => {
              this.brandOptions = data;
            });
          // 根据分类查询规格参数
          // this.$http
          //   .get("/item/spec/params?cid=" + this.goods.categories[2].id)
          //   .then(({ data }) => {
          //     let specs = [];
          //     let template = [];
          //     if (this.isEdit){
          //       specs = JSON.parse(this.goods.spuDetail.genericSpec);
          //       template = JSON.parse(this.goods.spuDetail.specialSpec);
          //     }
          //     // 对特有规格进行筛选
          //     const arr1 = [];
          //     const arr2 = [];
          //     data.forEach(({id, name,generic, numeric, unit }) => {
          //       if(generic){
          //         const o = { id, name, numeric, unit};
          //         if(this.isEdit){
          //           o.v = specs[id];
          //         }
          //         arr1.push(o)
          //       }else{
          //         const o = {id, name, options:[]};
          //         if(this.isEdit){
          //           o.options = template[id];
          //         }
          //         arr2.push(o)
          //       }
          //     });
          //     this.specs = arr1;// 通用规格
          //     this.specialSpecs = arr2;// 特有规格
          //   });
        }
      }
    }
  },
  computed: {
    skus() {
      // 过滤掉用户没有填写数据的规格参数
      const arr = this.specialSpecs.filter(s => s.options.length > 0);
      // 通过reduce进行累加笛卡尔积
      return arr.reduce(
        (last, spec, index) => {
          const result = [];
          last.forEach(o => {
            spec.options.forEach((option, i) => {
              const obj = JSON.parse(JSON.stringify(o));
              obj[spec.name] = {v:option, id:spec.id};
              obj.indexes = (obj.indexes || '') + '_' +  i
              if (index === arr.length - 1) {
                obj.indexes = obj.indexes.substring(1);
                // 如果发现是最后一组，则添加价格、库存等字段
                Object.assign(obj, {
                  price: 0,
                  stock: 0,
                  enable: false,
                  images: []
                });
                if (this.isEdit) {
                  // 如果是编辑，则回填sku信息
                  const sku = this.goods.skus.get(obj.indexes);
                  if (sku != null) {
                    const { price, stock, enable, images } = sku;
                    Object.assign(obj, {
                      price: this.$format(price),
                      stock,
                      enable,
                      images: images ? images.split(",") : [],
                    });
                  }
                }
              }
              result.push(obj);
            });
          });
          return result;
        },
        [{}]
      );
    },
    headers() {
      if (this.skus.length <= 0) {
        return [];
      }
      const headers = [];
      Object.keys(this.skus[0]).forEach(k => {
        let value = k;
        if (k === "price") {
          // enable，表头要翻译成“价格”
          k = "价格";
        } else if (k === "stock") {
          // enable，表头要翻译成“库存”
          k = "库存";
        } else if (k === "enable") {
          // enable，表头要翻译成“是否启用”
          k = "是否启用";
        } else if (k === "images" || k === 'indexes') {
          // 图片和索引不在表格中展示
          return;
        }
        headers.push({
          text: k,
          align: "center",
          sortable: false,
          value
        });
      });
      return headers;
    }
  }
};
</script>

<style scoped>
</style>
