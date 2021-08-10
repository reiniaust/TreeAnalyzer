<template>
  <div>
    <div>
      <DxTreeList
        :data-source="myData"
        :column-auto-width="true"
        :word-wrap-enabled="true"
        :show-borders="true"
        key-expr="id"
        parent-id-expr="parentId"
        @init-new-row="onInitNewRow"
        @saved="onSaved"
      >
        <DxEditing
          :allow-adding="true"
          :allow-updating="true"
          :allow-deleting="true"
          mode="row"
        />
        <DxSearchPanel
          :visible="true"
        />
        <DxFilterRow
          :visible="true"
        />
        <DxHeaderFilter
          :visible="true"
        />
        <DxColumn data-field="title" caption="Titel"> </DxColumn>
        <DxColumn data-field="linkId" caption="Verknüpfung">
          <DxLookup
            :data-source="myData"
            value-expr="id"
            display-expr="title"
          />
        </DxColumn>
        <DxColumn data-field="portion" caption="Anteil"> </DxColumn>
        <DxColumn data-field="portionUnitId" caption="Einheit">
          <DxLookup :data-source="units" value-expr="id" display-expr="title" />
        </DxColumn>
        <DxColumn data-field="total" caption="Gesamt"> </DxColumn>
        <DxColumn data-field="totalUnitId" caption="Einheit">
          <DxLookup :data-source="units" value-expr="id" display-expr="title" />
        </DxColumn>
        <DxColumn data-field="date" caption="Datum" data-type="date">
        </DxColumn>
        <DxColumn data-field="parentId" caption="Gehört zu">
          <DxLookup :data-source="myData" value-expr="id" display-expr="title" />
        </DxColumn>
      </DxTreeList>
    </div>
  </div>
</template>
<script>

import {
  DxTreeList,
  DxEditing,
  DxSearchPanel,
  DxColumn,
  DxLookup,
  DxFilterRow,
  DxHeaderFilter
} from "devextreme-vue/tree-list";

import deMessages from "devextreme/localization/messages/de.json";
import { locale, loadMessages } from "devextreme/localization";

export default {
  components: {
    DxTreeList,
    DxEditing,
    DxSearchPanel,
    DxColumn,
    DxLookup,
    DxFilterRow,
    DxHeaderFilter
  },
  data() {
    return {
      myData: [],
      units: []
    };
  },
  mounted() {
    loadMessages(deMessages);
    locale("de");

    try {
      this.myData = JSON.parse(localStorage.getItem('myTreeListData')).sort((a, b) => this.compare(a, b, "title"))
    } catch (error) {}

    if (!this.myData.find(d => d.title === "Einheiten")) {
      this.myData.push({id: 1, title: "Einheiten"})
      this.myData.push({id: 2, parentId: 1, title: "Min"})
      this.myData.push({id: 3, parentId: 1, title: "Std"})
      this.myData.push({id: 4, parentId: 1, title: "Tage"})
      this.myData.push({id: 5, parentId: 1, title: "Eur"})
      this.myData.push({id: 6, parentId: 1, title: "kg"})
      this.myData.push({id: 7, parentId: 1, title: "Stck"})
      this.myData.push({id: 8, parentId: 1, title: "Eur/Min"})
      this.myData.push({id: 9, parentId: 1, title: "Eur/Std"})
      this.myData.push({id: 10, parentId: 1, title: "Min/Stck"})
      this.myData.push({id: 11, parentId: 1, title: "kg/Stck"})
    }
    this.units = this.myData.filter(d => d.parentId === 1)
  },
  methods: {
    onInitNewRow(e) {
      // e.data.Task_Start_Date = new Date();
    },
    /*
    onEditorPreparing (e) {  
      if(e.parentType === 'dataRow'){  
        e.editorOptions.onValueChanged = this.setUnit
      } 
    }, */  
    onSaved(e) {
      this.myData.forEach((element) => {
        try {
          element.total = eval(element.portion)
        } catch (error) {}
        if (element.linkId) {
          let linkedRow = this.myData.find((d) => d.id === element.linkId)
          
          let linkedUnit = this.myData.find((d) => d.id === linkedRow.portionUnitId)
          if (linkedUnit) {
            let splitUnit = linkedUnit.title.split("/")
            if (splitUnit.length > 1) {
              element.portionUnitId = this.myData.find((d) => d.title === splitUnit[1]).id
              element.totalUnitId = this.myData.find((d) => d.title === splitUnit[0]).id
            }
          }

          let formular = linkedRow.portion;
          /*
          formular = formular.replace(" ", "")
          formular = formular.replace(",", ".")
          formular = formular.replace("x", element.portion)
          */
          formular += "*" + element.portion
          //alert(formular)
          element.total = Math.round(eval(formular)*100)/100
          //alert(formular)
        }
      });

      this.myData.filter(o => o.parentId === 0).forEach(element => {
        this.totalFromChidren(element)
      });

      localStorage.setItem('myTreeListData', JSON.stringify(this.myData));
    },
    totalFromChidren(parent) {
      let total = 0
      this.myData.filter(d => d.parentId === parent.id).forEach(child => {
        this.totalFromChidren(child)
        if (!parent.total) {
          parent.total = 0
        }

        // Ausschuss-Berechnung
        if (child.linkId) {
          if (this.myData.find((d) => d.id === child.linkId).title === "Ausschuss") {
            child.total = total * child.portion / 100  
          }
        }

        if (child.total) {
          parent.total += child.total
          total += child.total
        }
        
      });
    },
    compare(a, b, propName) {
      if (a[propName].toUpperCase() < b[propName].toUpperCase()) {
        return -1;
      }
      if (a[propName].toUpperCase() > b[propName].toUpperCase()) {
        return 1;
      }
      return 0;
    }  
  },
};
</script>

<style scoped>
</style>
