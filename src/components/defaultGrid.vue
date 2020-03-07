<template>
    <div style="width: 400px; height: 800px;">
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
                    :suppressRowClickSelection="true"
                    @cell-focused="cellFocused()">
        </ag-grid-vue>
    </div>
</template>

<script>
    import {AgGridVue} from "ag-grid-vue";

    export default {
        name: 'defaultGrid',
        data() {
            return {
                gridOptions: null,
                columnDefs: null,
                rowData: null,
                gridApi: null,
                defaultColDef:null,
                selectedColumn: null,
                selectedRow: null,
            }
        },

        components: {
            AgGridVue
        },

        methods:{
            cellFocused(){
                if(this.gridApi.getFocusedCell().column.colId === 'X'){
                    var selectedRows = this.gridApi.getSelectedNodes();

                    selectedRows.forEach(element => {
                        if(element.data.colType != 'int'){
                            element.setSelected(false);
                        }
                    });
                }

                this.selectedColumn = this.gridApi.getFocusedCell().column.colId;
                this.selectedRow = this.gridApi.getFocusedCell().rowIndex;
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
                }}
            ];

            this.rowData = [
                {colName:'VI', colType:'int', X:'', Y:''},
                {colName:'CO', colType:'int', X:'', Y:''},
                {colName:'AB', colType:'fac', X:'', Y:''},
                {colName:'FA', colType:'int', X:'', Y:''},
                {colName:'DI', colType:'fac', X:'', Y:''},
                {colName:'FN', colType:'cha', X:'', Y:''},
                {colName:'FS', colType:'cha', X:'', Y:''},
                {colName:'FC', colType:'cha', X:'', Y:''},
                {colName:'FZ', colType:'int', X:'', Y:''},
                {colName:'FS', colType:'int', X:'', Y:''},
                {colName:'CO', colType:'fac', X:'', Y:''},
                {colName:'DI', colType:'fac', X:'', Y:''},
                {colName:'CH', colType:'cha', X:'', Y:''}
            ];

            this.defaultColDef = {
                editable: true,
                resizable: true,
                cellStyle:{textAlign: 'left'},
                singleClickEdit:true
            }
        },

        mounted(){
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
            var allColumnIds = [];
            this.gridColumnApi.getAllColumns().forEach(function(column) {
                allColumnIds.push(column.colId);
            });
            this.gridColumnApi.autoSizeColumns(allColumnIds, false);
        }
    }
</script>