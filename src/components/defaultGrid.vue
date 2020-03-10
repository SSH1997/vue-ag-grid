<template>
    <div style="width: 700px; height: 800px;">
        <div id = 'cellTest' style="textAlign: left">
            Column : {{ this.selectedColumn }}<br/>
            Row : {{ this.selectedRow }}<br/>
            <button v-on:click="setColumnVisible(false)">컬럼 삭제</button>
            <button v-on:click="dataExtract()">데이터 확인</button>
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
        <div style="textAlign: left">
            Data :<br/>
            <ul>
                <li v-for="item in this.selectedData" v-bind:key="item">
                    {{ item }}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import {AgGridVue} from "ag-grid-vue";

    export default {
        name: 'defaultGrid',
        data(){
            return {
                gridOptions: null,
                columnDefs: null,
                rowData: null,
                gridApi: null,
                selectedColumn: null,
                selectedRow: null,
                selectedData: [],
            }
        },

        components: {
            AgGridVue
        },

        methods:{
            dataExtract(){
                this.selectedData = []; // 데이터 배열 초기화

                for(var i = 0; i < this.gridApi.getLastDisplayedRow(); i++){ // 보여지는 로우의 끝까지 검사
                    var row = this.gridApi.getDisplayedRowAtIndex(i);

                    if(document.getElementById("X"+i).checked == true && document.getElementById("Y"+i).checked == true){
                        row.data.X = 1;
                        row.data.Y = 1;
                    } // 둘다 체크 되어 있으면 x : 1 / y : 1
                    else if(document.getElementById("X"+i).checked == true){
                        row.data.X = 1;
                        row.data.Y = 0;
                    }
                    else if(document.getElementById("Y"+i).checked == true){
                        row.data.X = 0;
                        row.data.Y = 1;
                    }
                    else{
                        row.data.X = 0;
                        row.data.Y = 0;
                        continue;
                    } // 체크 안 되어 있으면 0, 0 및 데이터 배열에 추가 안함

                    row.data.colType = document.getElementById("colType" + i).value
                    this.selectedData.push(row.data); // 데이터 배열에 추가
                }
            },
            
            cellFocused(){
                var rowIndex = this.gridApi.getFocusedCell().rowIndex

                var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);

                row.setSelected(true);
            }, // 시각적 효과를 위해 cell에 포커스 되면 row가 선택되게 (흰색 -> 파란색)

            cellMouseOut(){
                var rowIndex = this.gridApi.getFocusedCell().rowIndex

                var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);

                row.setSelected(false);
            }, // 마우스를 cell에서 빼면 선택 해제 (파란색 -> 흰색)

            setColumnVisible(params){
                this.gridColumnApi.setColumnVisible(this.gridApi.getFocusedCell().column.colId,params)
            }, // 컬럼 삭제
            
            onGridReady(params) {
                params.api.sizeColumnsToFit();
            }
        },
        beforeMount() {
            this.gridOptions = {};
            this.columnDefs = [
                {headerName: '컬럼명', field: 'colName'},
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

                    return input
                }}
            ];

            this.rowData = [
                {colName:'VI', colType:'int', X:'0', Y:'0'},
                {colName:'CO', colType:'int', X:'0', Y:'0'},
                {colName:'AB', colType:'fac', X:'0', Y:'0'},
                {colName:'FA', colType:'int', X:'0', Y:'0'},
                {colName:'DI', colType:'fac', X:'0', Y:'0'},
                {colName:'FN', colType:'cha', X:'0', Y:'0'},
                {colName:'FS', colType:'cha', X:'0', Y:'0'},
                {colName:'FC', colType:'cha', X:'0', Y:'0'},
                {colName:'FZ', colType:'int', X:'0', Y:'0'},
                {colName:'FS', colType:'int', X:'0', Y:'0'},
                {colName:'CO', colType:'fac', X:'0', Y:'0'},
                {colName:'DI', colType:'fac', X:'0', Y:'0'},
                {colName:'CH', colType:'cha', X:'0', Y:'0'}
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
        }
    }
</script>