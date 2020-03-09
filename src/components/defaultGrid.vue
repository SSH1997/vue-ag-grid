<template>
    <div style="width: 700px; height: 800px;">
        <div id = 'cellTest' style="textAlign: left">
            Column : {{ this.selectedColumn }}<br/>
            Row : {{ this.selectedRow }}<br/>
            <button v-on:click="removeSelectedRows()">열 삭제</button>
            <button v-on:click="setColumnVisible(false)">컬럼 삭제</button>
        </div>
        <ag-grid-vue style="width: 100%; height: 50%;"
            class="ag-theme-balham"
            :defaultColDef="defaultColDef"
            @grid-ready="onGridReady"
            :gridOptions="gridOptions"
            :columnDefs="columnDefs"
            :rowData="rowData"
            rowSelection="multiple"
            :rowDeselection="true"
            :suppressRowClickSelection="true"
            :rowMultiSelectWithClick="true"
            @cell-focused="cellFocused()"
            @cell-clicked="cellClicked()"
            
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
                defaultColDef: null,
                selectedColumn: null,
                selectedRow: null,
                selectedData: [],
            }
        },
        components: {
            AgGridVue
        },
        methods:{
            cellFocused(){
                var rowIndex = this.gridApi.getFocusedCell().rowIndex

                var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);

                if(row.selected == true){
                    row.setSelected(false);
                    row.data.X = 0;
                    row.data.Y = 0;
                }
                
                else{
                    if(this.gridApi.getFocusedCell().column.colId === 'X'){
                        var selectedRows = this.gridApi.getSelectedNodes();
                        selectedRows.forEach(element => {
                            if(element.data.colType != 'int'){
                                element.setSelected(false);
                                element.data.X = 0;
                                element.data.Y = 0;
                            }
                        });
                    }

                    row.setSelected(true);
                }

                if(this.gridApi.getFocusedCell().column.colId === 'Y'){
                    row.setDataValue('count',Number(row.data.count) + (rowIndex + 1))
                }

                this.selectedColumn = this.gridApi.getFocusedCell().column.colId;
                this.selectedRow = this.gridApi.getFocusedCell().rowIndex;
                var selectedRows = this.gridApi.getSelectedNodes();

                this.selectedData=[];
                var temp = []
                selectedRows.forEach(element => {
                    if(element.data.colType == "cha" || element.data.colType == "fac"){
                        element.data.X = 1;
                    }

                    else if(element.data.colType == "int"){
                        element.data.Y = 1;
                    }

                    else{
                        element.data.X = 1;
                        element.data.Y = 1;
                    }

                    temp.push(element.data)

                    this.selectedData.push(temp)

                    temp = [];

                    this.gridApi.refreshCells();
                })
            },

            cellClikced(){
                var rowIndex = this.gridApi.getFocusedCell().rowIndex

                var row = this.gridApi.getDisplayedRowAtIndex(rowIndex);
            },

            removeSelectedRows(){
                var selectedRows = this.gridApi.getSelectedRows();

                this.gridApi.updateRowData({remove:selectedRows})
            },

            setColumnVisible(params){
                this.gridColumnApi.setColumnVisible(this.gridApi.getFocusedCell().column.colId,params)
            },
            
            onGridReady(params) {
                params.api.sizeColumnsToFit();
            }
        },
        beforeMount() {
            this.gridOptions = {};

            /*this.columnDefs = [
                {headerName: 'Name', field: 'name'},
                {headerName: 'Age', field: 'age',cellStyle: { backgroundColor: "#aaffaa" }, checkboxSelection : function(params){
                    if(params.data.age > "25")
                        return true
                }},
                {headerName: 'Birthday', field: 'birthday',checkboxSelection : function(params){
                    if(params.data.age <= "25")
                        return true
                }},
                {headerName: 'Gender', field: 'gender',cellEditor: "agSelectCellEditor",
                    cellEditorParams: {cellHeight: 50, values: ["male", "female"]}},
                {headerName: 'Hobby', field: 'hobby'}
            ];

            this.rowData = [
                {name: 'Sam', age: '30', birthday: '00/00/00', gender: 'male', hobby:'playing piano'},
                {name: 'Ariana', age: '25', birthday: '00/00/00', gender: 'female', hobby:'soccer'},
                {name: 'Elsa', age: '20', birthday: '00/00/00', gender: 'female', hobby:'watching youtube'},
                {name: 'Steve', age: '24', birthday: '00/00/00', gender: 'male', hobby:'netflix'},
                {name: 'Tom', age: '34', birthday: '00/00/00', gender: 'male', hobby:'cooking'}
            ];*/

            this.columnDefs = [
                {headerName: '컬럼명', field: 'colName'},
                {headerName: '자료형식', field: 'colType'},
                {headerName: 'X', field: 'X', checkboxSelection: function (params) {
                    if(params.data.colType == "cha" || params.data.colType == "fac")
                        return true
                }},
                {headerName: 'Y', field: 'Y', checkboxSelection: function (params) {
                    if(params.data.colType == "int")
                        return true
                }},
                {headerName: 'Count', field: 'count'},
            ];

            this.rowData = [
                {colName:'VI', colType:'int', X:'0', Y:'0', count:0},
                {colName:'CO', colType:'int', X:'0', Y:'0', count:0},
                {colName:'AB', colType:'fac', X:'0', Y:'0', count:0},
                {colName:'FA', colType:'int', X:'0', Y:'0', count:0},
                {colName:'DI', colType:'fac', X:'0', Y:'0', count:0},
                {colName:'FN', colType:'cha', X:'0', Y:'0', count:0},
                {colName:'FS', colType:'cha', X:'0', Y:'0', count:0},
                {colName:'FC', colType:'cha', X:'0', Y:'0', count:0},
                {colName:'FZ', colType:'int', X:'0', Y:'0', count:0},
                {colName:'FS', colType:'int', X:'0', Y:'0', count:0},
                {colName:'CO', colType:'fac', X:'0', Y:'0', count:0},
                {colName:'DI', colType:'fac', X:'0', Y:'0', count:0},
                {colName:'CH', colType:'cha', X:'0', Y:'0', count:0}
            ];

            this.defaultColDef = {
                editable: true,
                resizable: true,
                cellStyle:{textAlign: 'left'},
                singleClickEdit:true,
            }
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

<!-- <ag-grid-column headerName="IT Skills">
            <ag-grid-column field="skills" :width="120" suppressSorting
                            cellRendererFramework="SkillsCellRenderer"
                            :menuTabs="['filterMenuTab']">
            </ag-grid-column>
            <ag-grid-column field="proficiency" :width="135"
                            cellRendererFramework="ProficiencyCellRenderer"
                            :menuTabs="['filterMenuTab']">
            </ag-grid-column>
        </ag-grid-column> -->