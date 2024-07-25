<template>
    <div id="app">
        <ejs-grid ref="grid" :dataSource='data' :toolbar="toolbar" :editSettings="editSettings" allowSorting="true" allowPaging="true" :created="created" :actionComplete="actionComplete">
            <e-columns>
                <e-column field='OrderID' headerText='Order ID' width='120' textAlign='Right' isPrimaryKey="true"></e-column>
                <e-column field='CustomerID' headerText='Customer ID' width='160'></e-column>
                <e-column field='Freight' headerText='Freight' format="C" width='150'></e-column>
                <e-column field='ShipCity' headerText='Ship City' width='150'></e-column>
                <e-column field='ShipCountry' headerText='Ship Country' width='150'></e-column>
            </e-columns>
        </ejs-grid>
    </div>
</template>
<script setup>
    import { provide, ref } from "vue";
    import { GridComponent as EjsGrid, ColumnDirective as EColumn, ColumnsDirective as EColumns, Page, Edit, Sort, Filter, Toolbar } from "@syncfusion/ej2-vue-grids";
    import { DataManager, UrlAdaptor } from "@syncfusion/ej2-data";
    import * as signalR from '@microsoft/signalr';
    const grid = ref(null);

    const data = new DataManager({
        url: 'https://localhost:7231/api/Home',
        insertUrl: 'https://localhost:7231/api/Home/Insert',
        updateUrl: 'https://localhost:7231/api/Home/Update',
        removeUrl: 'https://localhost:7231/api/Home/Remove',
        adaptor: new UrlAdaptor(),
    });

    let connection = new signalR.HubConnectionBuilder().withUrl("https://localhost:7231/ChatHub").build();
    const editSettings = { allowEditing: true, allowAdding: true, allowDeleting: true };
    const toolbar = ['Add', 'Edit', 'Update', 'Delete', 'Search'];
    const created = function () {


        connection.on("ReceiveMessage", (message) => {
            if (grid.value) {
                grid.value.ej2Instances.refresh();
            }

        });

        connection.start()
            .then(() => {
                console.log("SignalR connection established successfully");
                connection.invoke('SendMessage', "refreshPages")
                    .catch((err) => {
                        console.error("Error sending data:", err.toString());
                    });
            })
            .catch((err) => {
                console.error("Error establishing SignalR connection:", err.toString());
            });
    };
    const actionComplete = function (args) {
        if (args.requestType === "save" || args.requestType === "delete") {
            connection.invoke('SendMessage', "refreshPages")
                .catch((err) => {
                    console.error(err.toString());
                });
        }
    }
    provide('grid', [Sort, Edit, Filter, Page, Toolbar]);

</script>
<style>
    @import "../node_modules/@syncfusion/ej2-base/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-buttons/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-calendars/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-dropdowns/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-inputs/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-navigations/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-popups/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-splitbuttons/styles/tailwind.css";
    @import "../node_modules/@syncfusion/ej2-vue-grids/styles/tailwind.css";
</style>