<template>
    <div style="width: 700px; height: 800px;">
        <div ref="menuButton" class="customHeaderMenuButton"></div>
        <div id = 'cellTest' style="textAlign: left">
            <button v-on:click="dataSave()">데이터 저장</button>
        </div>
        <ag-grid-vue style="width: 100%; height: 50%;"
            class="ag-theme-balham"
            @grid-ready="onGridReady"
            :gridOptions="gridOptions"
            :columnDefs="columnDefs"
            :rowData="rowData"
            rowSelection="multiple"
            :rowDeselection="true"
            :suppressRowClickSelection="true"
            :rowMultiSelectWithClick="true"
            @cell-focused="cellFocused()"
            @cell-mouse-out="cellMouseOut()"
            :enterMovesDownAfterEdit="true"
            :enterMovesDown="true">
        </ag-grid-vue>
    </div>
</template>

<script>
    import {AgGridVue} from "ag-grid-vue";
    import rowDataSet from "../assets/rowData.json"
    import tagSet from "../assets/tagSet.json"
    import { EventBus } from './event-bus.js'

    export default {
        name: 'defaultGrid',
        props: ['tabId'],
        data(){
            return {
                gridOptions: null,
                columnDefs: null,
                gridApi: null,
                shiftStart:null,
                shiftEnd:null,

                checkedDepVar: null,
                checkedIndepVar: new Array()
            }
        },

        components: {
            AgGridVue
        },

        created () {
            EventBus.$on("getOptions", () => {
                console.log("received")

                this.getOptionChecked()

                let payload = {
                    tabId: this.tabId,
                    indepVar: this.checkedIndepVar,
                    depVar: this.checkedDepVar
                }

                EventBus.$emit("returnOptions", payload)
            })
        },

        methods:{
            dataSave(){
                /* for(var i = 0; i < this.gridApi.getLastDisplayedRow(); i++){ // 보여지는 로우의 끝까지 검사
                    if(document.getElementById("X"+i).checked){
                        this.rowData[i].X = 1
                    }
                    else{
                        this.rowData[i].X = 0
                    }

                    if(document.getElementById("Y"+i).checked){
                        this.rowData[i].Y = 1
                    }
                    else{
                        this.rowData[i].Y = 0
                    }

                    this.rowData[i].colType = document.getElementById('colType' + i).value
                } */
            },
            
            cellFocused(){  
                var rowIndex = this.gridApi.getFocusedCell().rowIndex

                var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);

                var selectedRows = this.gridApi.getSelectedNodes();

                selectedRows.forEach(element => {
                    element.setSelected(false);   
                });

                row.setSelected(true);
            }, // 시각적 효과를 위해 cell에 포커스 되면 row가 선택되게 (흰색 -> 파란색)

            cellMouseOut(){
                if(this.gridApi.getFocusedCell() != null){
                     var rowIndex = this.gridApi.getFocusedCell().rowIndex
                
                    var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);
                
                    row.setSelected(false);
                }
            }, // 마우스를 cell에서 빼면 선택 해제 (파란색 -> 흰색)
            
            onGridReady(params) {
                params.api.sizeColumnsToFit();
                for(var i = 0; i < this.gridApi.getLastDisplayedRow(); i++){ // 보여지는 로우의 끝까지 검사
                    var row = this.gridApi.getDisplayedRowAtIndex(i);
                    if(row.data.X == 1){
                        document.getElementById("X"+i).checked = true;
                    }
                    if(row.data.Y == 1){
                        document.getElementById("Y"+i).checked = true;
                    }
                }

                this.insertTags()
            },

            insertTags(){
                for(var i = 0; i < this.rowData.length; i++){
                    this.tags.forEach(element => {
                        if(this.rowData[i].colName == element.target){
                            document.getElementById('tag' + i).innerHTML = element.tag
                        }
                    });
                }
                console.log(document.getElementById('tag0'))
            },

            getOptionChecked () {
                this.checkedDepVar = null;
                this.checkedIndepVar = [];
                
                if(window.localStorage.getItem("X" + this.tabId) != null){
                    var tempX = window.localStorage.getItem("X" + this.tabId).split(",")
                    var tempY = window.localStorage.getItem("Y" + this.tabId).split(",")

                    for(var i=0; i<this.rowData.length; i++) {
                        if(tempX[i] == "1"){
                            this.checkedDepVar = this.rowData[i].colName
                        }
                        if(tempY[i] == "1"){
                            this.checkedIndepVar.push(this.rowData[i].colName)
                        }
                    }
                }
            }
        },

        beforeMount() {
            this.gridOptions = {};
            this.tags = [
                {
                    target:'VI',
                    tag:['tag1','tag2','tag3']
                },
                {
                    target:'AB',
                    tag:['tag3','tag4','tag5']
                },
                {
                    target:'FN',
                    tag:['tag6','tag7','tag8']
                },
                {
                    target:'FS',
                    tag:['tag9','tag0','tag1']
                },
            ]
            this.columnDefs = [
                {headerName: '컬럼명', field: 'colName', cellRenderer: function(params){
                    var cellDiv = document.createElement("div")
                    var dataDiv = document.createElement("div")
                    var tagDiv = document.createElement("div")
                    
                    //cellDiv.setAttribute("style","layout:null;")

                    dataDiv.innerHTML = params.value
                    dataDiv.setAttribute("style","position:absolute; left:50%; transform:translate(-50%) width:15px; float:left")

                    tagDiv.setAttribute("id","tag" + params.rowIndex)

                    tagDiv.setAttribute("style", 'position:absolute; right:2%; background-color:lightgray;float:left')
                    
                    cellDiv.appendChild(dataDiv)
                    cellDiv.appendChild(tagDiv)

                    return cellDiv
                }},
                {headerName: '자료형식', field: 'colType', cellRenderer: function(params){
                    var select = document.createElement("select") // select 생성

                    var dataArray = ["int","cha","fac"] // 실제 데이터
                    var textArray = ["정수","문자","요소"]  // 보여지는 값

                    for(var i = 0 ; i < dataArray.length; i++){ // option 반복 생성 및 할당
                        var option = document.createElement("option")

                        option.value = dataArray[i] // 실제 데이터 할당
                        option.text = textArray[i] // 보여지는 값 할당

                        if(params.node.data.colType == dataArray[i]){
                            option.defaultSelected=true; // 초기값 설정
                        }

                        select.appendChild(option)
                    }

                    select.id = "colType" + params.rowIndex;

                    return select
                }},
                {headerName: 'X', field: 'X', cellRenderer: function(params) {
                    var input = document.createElement("input")
                    input.type = "checkbox"
                    input.checked = false;
                    input.id = "X" + params.rowIndex; // 로우 index에 맞추어 id를 설정

                    input.addEventListener('click', function (event) { // 클릭 이벤트 리스너

                        for(var i = 0; i < params.node.parent.allChildrenCount; i++){ // 부모 노드가 가지고 있는 자식 노드의 갯수 가져와서 모든 노드 검사
                            if(params.rowIndex == i){
                                continue;
                            }
                            if(document.getElementById("X"+i).checked == true){
                                document.getElementById("X"+i).checked = false
                            }
                        }
                    });

                    return input
                }},
                {headerName: 'Y', field: 'Y',cellRenderer: function(params) {
                    var input = document.createElement("input")
                    input.type = "checkbox"
                    input.checked = false;
                    input.id = "Y" + params.rowIndex;
                    window.localStorage.removeItem('shiftStart');
                    window.localStorage.removeItem('shiftEnd');
                    input.addEventListener('click', function (event) {
                        if(event.shiftKey){
                            if(window.localStorage.getItem('shiftStart') == null){
                                window.localStorage.setItem('shiftStart', params.rowIndex)
                            }
                            else{
                                window.localStorage.setItem('shiftEnd', params.rowIndex)

                                if(window.localStorage.getItem('shiftStart') > params.rowIndex){
                                    window.localStorage.setItem('shiftEnd', window.localStorage.getItem('shiftStart'))    
                                    window.localStorage.setItem('shiftStart', params.rowIndex)
                                }

                                for(var i = Number(window.localStorage.getItem('shiftStart')); i <= Number(window.localStorage.getItem('shiftEnd')); i++){
                                    document.getElementById("Y"+i).checked = true;
                                }
                                window.localStorage.removeItem('shiftStart');
                                window.localStorage.removeItem('shiftEnd');
                            }
                        }
                    })

                    return input
                }}
            ];
        },

        mounted(){
            this.gridApi = this.gridOptions.api;
            var allColumnIds = [];
            this.gridColumnApi = this.gridOptions.columnApi;
            this.gridColumnApi.getAllColumns().forEach(function(column) {
                allColumnIds.push(column.colId);
            });
            this.gridColumnApi.autoSizeColumns(allColumnIds, false);
        },
        beforeDestroy(){
            window.localStorage.clear()
        },
        computed:{
            rowData() {
                return rowDataSet.rowData 
            },

            tags(){
                return tagSet.tagSet
            }
        }
    }
</script>