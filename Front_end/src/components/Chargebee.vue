<template>
    <div class="all-member-tbl" v-if="visibleFlag && dataFlag">
        <h2>All Member table</h2>
        <mdb-datatable-2
            v-model="tbl_data"
        />
    </div>
</template>

<script>
import { mdbDatatable2 } from 'mdbvue';

export default {
    name: 'Chargebee',
    components: {
        mdbDatatable2
    },
    data () {
        return {
            tbl_data: {
                columns: [
                    {
                      label: 'Email',
                      field: 'id',
                      sort: true
                    },
                    {
                      label: 'Recurring Items',
                      field: 'recurring',
                      sort: true
                    },
                    {
                      label: 'Status',
                      field: 'status',
                      sort: true
                    },
                    {
                      label: 'Created At',
                      field: 'created_at',
                      sort: true
                    },
                    {
                      label: 'Next Billing At',
                      field: 'next_billing',
                      sort: true
                    }
                ],
                rows: null
            },
            dataFlag: false
        }
    },
    props: {
        data: {
            type: Array,
            required: false,
            default: null
        },
        visibleFlag: {
            type: Boolean,
            required: false,
            default: false
        }
    },
    updated () {
        this.tbl_data.rows = this.data;
        this.dataFlag = true;
    }
};
</script>