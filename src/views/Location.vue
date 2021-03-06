<template>
  <div class="main-layout-details">
    <div class="card">
      <DataTable
        :rowHover="true"
        :scrollable="true"
        ref="dt"
        :value="getLocationList"
        dataKey="id"
        :paginator="true"
        :rows="5"
        v-model:filters="filters"
        :globalFilterFields="['name', 'description']"
        filterDisplay="menu"
        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
        currentPageReportTemplate=""
      >
        <div class="table-header">
          <h3
            class="p-m-0"
            style="font-size:22px; color: rgb(20, 49, 120); font-weight: 400"
          >
            Areas
          </h3>
          <span class="p-input-icon-left">
            <Button
              icon="pi pi-plus"
              class="p-button-info p-mr-2"
              @click="openNew"
              style="margin:2px"
              label="New"
              v-if="admin"
            />
            <span class="p-input-icon-left" style="margin:2px">
              <i class="pi pi-search" />
              <InputText
                v-model="filters['global'].value"
                placeholder="Keyword Search"
              />
            </span>
          </span>
        </div>

        <template #empty>
          No Areas found.
        </template>
        <Column
          header="No"
          style="margin-right: -10rem"
          headerStyle="width: 30px;border-radius: 20px 0 0 20px"
        >
          <template #body="slotProps">
            {{ slotProps.data.index }}
          </template>
        </Column>
        <Column
          field="name"
          header="Area Name"
          :showFilterMatchModes="false"
          :showAddButton="false"
          :showFilterOperator="false"
        >
          <template #body="slotProps">
            {{ slotProps.data.name }}
          </template>
          <template #filter="{filterModel}">
            <InputText
              type="text"
              v-model="filterModel.value"
              class="p-column-filter"
              placeholder="Search by name"
            />
          </template>
        </Column>
        <Column
          field="description"
          header="Description"
          :showFilterMatchModes="false"
          :showAddButton="false"
          :showFilterOperator="false"
        >
          <template #body="slotProps">
            <span
              v-if="
                slotProps.data.description !== '' &&
                  slotProps.data.description !== null
              "
              >{{ slotProps.data.description }}</span
            >
            <span v-else style="font-style: italic; color: #adadad"
              >No description</span
            >
          </template>
          <template #filter="{filterModel}">
            <InputText
              type="text"
              v-model="filterModel.value"
              class="p-column-filter"
              placeholder="Search by Description"
            />
          </template>
        </Column>
        <Column
          header="Created"
          filterField="created"
          dataType="date"
          :showAddButton="false"
          :showFilterOperator="false"
        >
          <template #body="{data}">
            {{ callDate(data.created) }}
          </template>
          <template #filter="{filterModel}">
            <Calendar
              v-model="filterModel.value"
              dateFormat="dd/mm/yy"
              placeholder="dd/mm/yyyy"
            />
          </template>
        </Column>
        <Column
          header="Last Modified"
          filterField="lastModified"
          dataType="date"
          :showAddButton="false"
          :showFilterOperator="false"
        >
          <template #body="{data}">
            {{ callDate(data.lastModified) }}
          </template>
          <template #filter="{filterModel}">
            <Calendar
              v-model="filterModel.value"
              dateFormat="dd/mm/yy"
              placeholder="dd/mm/yyyy"
            />
          </template>
        </Column>
        <Column headerStyle="border-radius:0 20px 20px 0">
          <template #body="slotProps">
            <Button
              icon="pi pi-pencil"
              class="p-button-rounded p-button-info p-button-text p-mr-2"
              @click="editProduct(slotProps.data)"
              v-tooltip.bottom="'Edit'"
              style="margin: 2px"
              v-if="admin"
            />
            <Button
              icon="pi pi-eye"
              class="p-button-rounded p-button-info p-button-text p-mr-2"
              @click="editProduct(slotProps.data)"
              v-tooltip.bottom="'View Area Details'"
              style="margin: 2px"
              v-else
            />
            <Button
              icon="pi pi-trash"
              class="p-button-rounded p-button-danger p-button-text"
              @click="confirmDeleteProduct(slotProps.data)"
              style="margin: 2px"
              v-tooltip.bottom="'Delete Area'"
              v-if="admin"
            />
          </template>
        </Column>
      </DataTable>
    </div>
    <Dialog
      v-model:visible="createLocationDiaglog"
      :style="{ width: '450px' }"
      header="Area Information"
      :modal="true"
      class="p-fluid"
    >
      <div class="p-field">
        <label for="name" class="form-control-label">Area Name</label>
        <InputText
          id="locationName"
          v-model.trim="locationName"
          maxlength="30"
          class="form-control form-control-alternative"
          placeholder="Area Name"
        />
        <small class="invalid">{{ errors.locationName }}</small>
      </div>
      <div class="p-field">
        <label for="description" class="form-control-label"
          >Description (optional)</label
        >
        <Textarea
          v-model.trim="description"
          class="form-control form-control-alternative"
          rows="3"
          cols="20"
          style="width:100%; height: auto;"
          placeholder="Description"
          :autoResize="true"
        />
        <small class="invalid">{{ errors.description }}</small>
      </div>
      <template #footer>
        <Button
          label="Cancel"
          icon="pi pi-times"
          class="p-button-text"
          @click="hideDialog"
        />
        <Button
          label="Create"
          icon="pi pi-check"
          class="p-button-text"
          @click="createLocation"
        />
      </template>
    </Dialog>
    <Dialog
      v-model:visible="productDialog"
      :style="{ width: '450px' }"
      header="Area Information"
      :modal="true"
      class="p-fluid"
    >
      <div class="p-field">
        <label for="name" class="form-control-label">Area Name</label>
        <InputText
          id="locationName"
          v-model.trim="locationName"
          required="true"
          maxlength="30"
          :readonly="!admin"
          class="form-control form-control-alternative"
          placeholder="Description"
        />
        <small class="invalid"> {{ errors.locationName }}</small>
      </div>
      <div class="p-field">
        <label for="description" class="form-control-label"
          >Description (optional)</label
        >
        <Textarea
          v-model.trim="description"
          class="form-control form-control-alternative"
          rows="3"
          cols="20"
          style="width:100%; height: auto;"
          placeholder="Description"
          :readonly="!admin"
          :autoResize="true"
        />
        <small class="invalid"> {{ errors.description }}</small>
      </div>
      <div class="p-formgrid p-grid">
        <div class="p-field p-col-6">
          <label for="created" class="form-control-label"> Created Date</label>
          <InputText
            v-model.trim="product.created"
            readonly
            class="form-control form-control-alternative"
          />
        </div>
        <div class="p-field p-col-6">
          <label for="lastModified" class="form-control-label">
            Last Modified</label
          >
          <InputText
            v-model.trim="product.lastModified"
            readonly
            class="form-control form-control-alternative"
          />
        </div>
      </div>
      <template #footer>
        <Button
          label="Cancel"
          icon="pi pi-times"
          class="p-button-text"
          @click="hideDialog"
        />
        <Button
          label="Save"
          icon="pi pi-check"
          class="p-button-text"
          v-if="admin"
          @click="updateLocation"
        />
      </template>
    </Dialog>
    <Dialog
      v-model:visible="deleteProductsDialog"
      :style="{ width: '450px' }"
      header="Confirm"
      :modal="true"
    >
      <div class="confirmation-content">
        <i class="pi pi-exclamation-triangle p-mr-3" style="font-size: 2rem" />
        <span v-if="product"
          >Are you sure to delete the area
          <span style="color:red; font-weight: bold;">{{ product.name }}</span
          >?</span
        >
      </div>
      <template #footer>
        <Button
          label="No"
          icon="pi pi-times"
          class="p-button-text"
          @click="deleteProductsDialog = false"
        />
        <Button
          label="Yes"
          icon="pi pi-check"
          class="p-button-text"
          @click="deleteSelectedLocation"
        />
      </template>
    </Dialog>
    <Toast position="bottom-right" />
  </div>
</template>

<script>
import Toast from "primevue/toast";
import Button from "primevue/button";
import Calendar from "primevue/calendar";
import { locationApi } from "../apis/location";
import moment from "moment";
import contentNoti from "../util/contentNoti.js";
import { mapGetters, mapActions } from "vuex";
import { useForm, useField } from "vee-validate";
import { FilterMatchMode, FilterOperator } from "primevue/api";
import webRole from "../util/webRole.js";
import * as yup from "yup";

export default {
  setup() {
    const schema = yup.object({
      locationName: yup
        .string()
        .label("Area Name")
        .required()
        .max(20),
      description: yup
        .string()
        .label("Description")
        .max(100)
        .nullable(),
    });

    const { errors, meta, handleReset, validate } = useForm({
      validationSchema: schema,
    });

    const { value: description } = useField("description");
    const { value: locationName } = useField("locationName");

    return {
      description,
      locationName,
      errors,
      meta,
      handleReset,
      validate,
    };
  },

  components: {
    Button,
    Calendar,
    Toast,
  },
  computed: {
    ...mapGetters("location", ["getLocationList"]),

    data() {
      return this.getLocationList;
    },
  },
  data() {
    return {
      deleteProductsDialog: false,
      productDialog: false,
      createLocationDiaglog: false,
      product: {},
      filters: {},
      submitted: false,
      messages: [],
      warnning: "",
      search: false,
      loading: true,
      role: null,
      admin: false,
    };
  },
  created() {
    this.initFilters();
    this.setLocationList();
    this.loading = false;
  },

  mounted() {
    this.role = JSON.parse(localStorage.getItem("user")).role;
    if (this.role === webRole.ADMIN_ROLE) {
      this.admin = true;
    }
  },
  methods: {
    ...mapActions("location", ["setLocationList"]),

    confirmDeleteProduct(product) {
      this.product = product;
      this.deleteProductsDialog = true;
    },
    openNew() {
      this.handleReset();
      this.createLocationDiaglog = true;
    },
    async createLocation() {
      if (this.meta.valid && this.locationName != null) {
        await locationApi
          .create(this.locationName, this.description)
          .then((res) => {
            if (res.status == 200) {
              this.$toast.add({
                severity: "success",
                summary: contentNoti.SUCCESS_SUMMARY,
                detail: contentNoti.LOCATION_CREATE_SUCCESS,
                life: 3000,
              });
              this.setLocationList();
              this.submitted = false;
              this.hideDialog();
            } else {
              this.$toast.add({
                severity: "error",
                summary: contentNoti.FAIL_SUMMARY,
                detail: contentNoti.LOCATION_CREATE_FAILED,
                life: 3000,
              });
              this.ChangePassworDialog = false;
            }
          })
          .catch(() => {
            this.$toast.add({
              severity: "error",
              summary: contentNoti.FAIL_SUMMARY,
              detail: contentNoti.LOCATION_CREATE_FAILED,
              life: 3000,
            });
            this.ChangePassworDialog = false;
          });
      } else {
        this.validate();
      }
    },
    editProduct(product) {
      this.handleReset();
      this.product = { ...product };
      this.locationName = this.product.name;
      this.description = this.product.description;
      this.product.created = this.callDate(product.created);
      this.product.lastModified = this.callDate(product.lastModified);
      this.submitted = false;
      this.productDialog = true;
    },
    hideDialog() {
      this.createLocationDiaglog = false;
      this.productDialog = false;
      this.submitted = false;
      this.product = {};
      this.deleteProductsDialog = false;
    },
    showAssessmentDialog(product) {
      this.product = { ...product };
      this.showAssessment = true;
    },
    async deleteSelectedLocation() {
      await locationApi
        .disable(this.product.locationId)
        .then((res) => {
          if (res.status != 200) {
            this.$toast.add({
              severity: "error",
              summary: contentNoti.FAIL_SUMMARY,
              detail: contentNoti.LOCATION_DISABLE_FAILED,
              life: 3000,
            });
            this.hideDialog();
          } else {
            this.$toast.add({
              severity: "success",
              summary: contentNoti.SUCCESS_SUMMARY,
              detail: contentNoti.LOCATION_DISABLE_SUCCESS,
              life: 3000,
            });
            this.setLocationList();
            this.hideDialog();
          }
        })
        .catch(() => {
          this.$toast.add({
            severity: "error",
            summary: contentNoti.FAIL_SUMMARY,
            detail: contentNoti.LOCATION_DISABLE_FAILED,
            life: 3000,
          });
          this.hideDialog();
        });
    },
    async updateLocation() {
      if (this.meta.valid) {
        await locationApi
          .update(this.product.locationId, this.locationName, this.description)
          .then((res) => {
            if (res.status == 200) {
              this.$toast.add({
                severity: "success",
                summary: contentNoti.SUCCESS_SUMMARY,
                detail: contentNoti.LOCATION_EDIT_SUCCESS,
                life: 3000,
              });
              this.setLocationList();
              this.productDialog = false;
            } else {
              this.$toast.add({
                severity: "error",
                summary: contentNoti.FAIL_SUMMARY,
                detail: contentNoti.LOCATION_EDIT_FAILED,
                life: 3000,
              });
              this.productDialog = false;
            }
          })
          .catch(() => {
            this.$toast.add({
              severity: "error",
              summary: contentNoti.FAIL_SUMMARY,
              detail: contentNoti.LOCATION_EDIT_FAILED,
              life: 3000,
            });
            this.productDialog = false;
          });
      } else {
        this.validate();
      }
    },
    findIndexById(id) {
      let index = -1;
      for (let i = 0; i < this.getLocationList.length; i++) {
        if (this.getLocationList[i].id === id) {
          index = i;
          break;
        }
      }
      return index;
    },
    callDate(date) {
      const date1 = new Date(date);
      return moment(date1).format("DD/MM/YYYY HH:mm:ss");
    },
    filterDate(value, filter) {
      if (
        filter === undefined ||
        filter === null ||
        (typeof filter === "string" && filter.trim() === "")
      ) {
        return true;
      }

      if (value === undefined || value === null) {
        return false;
      }
      let tmp = this.callDate(value).substring(0, 10);
      return tmp === this.formatDate(filter);
    },
    formatDateTMP(date) {
      let month = date.getMonth() + 1;
      let day = date.getDate();
      if (month < 10) {
        month = "0" + month;
      }
      if (day < 10) {
        day = "0" + day;
      }
      return day + "-" + month + "-" + date.getFullYear();
    },
    formatDate(value) {
      return new Date(value).toLocaleDateString("en-US", {
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      });
    },
    initFilters() {
      this.filters = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: {
          operator: FilterOperator.AND,
          constraints: [{ value: null, matchMode: FilterMatchMode.CONTAINS }],
        },
        description: {
          operator: FilterOperator.AND,
          constraints: [{ value: null, matchMode: FilterMatchMode.CONTAINS }],
        },
        created: {
          operator: FilterOperator.AND,
          constraints: [{ value: null, matchMode: FilterMatchMode.DATE_IS }],
        },
        lastModified: {
          operator: FilterOperator.AND,
          constraints: [{ value: null, matchMode: FilterMatchMode.DATE_IS }],
        },
      };
    },
  },
};
</script>

<style scoped>
label {
  display: inline-block;
  margin-bottom: 0.5rem;
}
.form-control-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #525f7f;
}
.form-control:-ms-input-placeholder {
  opacity: 1;
  color: #adb5bd;
}
.form-control::-ms-input-placeholder {
  opacity: 1;
  color: #adb5bd;
}
.form-control::placeholder {
  opacity: 1;
  color: #adb5bd;
}
.form-control:focus:-ms-input-placeholder {
  color: #adb5bd;
}
.form-control:focus::-ms-input-placeholder {
  color: #adb5bd;
}
.form-control:focus::placeholder {
  color: #adb5bd;
}
.form-control {
  font-size: 1rem;
  line-height: 1.5;
  display: block;
  width: 100%;
  height: calc(2.75rem + 2px);
  padding: 0.625rem 0.75rem;
  transition: all 0.2s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  color: #8898aa;
  border: 1px solid #cad1d7;
  border-radius: 0.375rem;
  background-color: #fff;
  background-clip: padding-box;
  box-shadow: none;
}
.form-control:focus {
  color: #8898aa;
  border-color: rgba(50, 151, 211, 0.25);
  outline: 0;
  background-color: #fff;
  box-shadow: none, none;
}
.form-control {
  font-size: 0.875rem;
}
.focused .form-control {
  border-color: rgba(50, 151, 211, 0.25);
}
.form-control:disabled,
.form-control[readonly] {
  opacity: 1;
  background-color: #ebf0f5;
}
.form-control-alternative {
  transition: box-shadow 0.15s ease;
  border: 0;
  box-shadow: 0 1px 3px rgba(50, 50, 93, 0.15), 0 1px 0 rgba(0, 0, 0, 0.02);
}
.form-control-alternative:focus {
  box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11), 0 1px 3px rgba(0, 0, 0, 0.08);
}
.table-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.08);
  border-width: 0 0 0 0;
  color: #69707a;
  padding: 1rem;
  font-weight: 700;
  border-top-left-radius: 24px;
  border-top-right-radius: 24px;
}

.product-image {
  width: 100px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}

.p-dialog .product-image {
  width: 150px;
  margin: 0 auto 2rem auto;
  display: block;
}

.confirmation-content {
  display: flex;
  align-items: center;
}
.invalid {
  color: red;
  position: sticky;
  left: 5px !important;
}

.p-m-0 {
  display: flex;
  align-items: center;
  justify-content: center;
}

.p-m-0::before {
  content: "";
  width: 5px;
  height: 12px;
  display: block;
  border-radius: 3px;
  padding-bottom: 10px;
  position: relative;
  left: -10px;
  top: 0;
  background: #007dfe;
}

::v-deep(.p-dialog .p-dialog-header .p-dialog-title::before) {
  content: "";
  width: 5px;
  height: 12px;
  display: block;
  border-radius: 3px;
  padding-bottom: 10px;
  position: relative;
  left: -10px;
  top: 0;
  background: #007dfe;
}

::v-deep(.p-datatable .p-datatable-thead > tr > th) {
  background: #f2f9ff;
  color: #4ca4f9;
  padding: 1rem;
  border: 1px solid rgba(0, 0, 0, 0.08);
  border-width: 0 0 0 0;
  text-align: left;
  box-sizing: content-box;
  transition: background-color 0.2s, color 0.2s, border-color 0.2s,
    box-shadow 0.2s;
  cursor: pointer;
  font-weight: 500;
  display: flex;
  justify-content: left;
  align-items: center;
}
::v-deep(.p-datatable .p-datatable-tbody > tr > td) {
  text-align: left;
  border: 1px solid #e9ecef;
  border-width: 0 0 0 0;
  padding: 1rem 1rem;
  display: flex;
  justify-content: left;
  align-items: center;
}

::v-deep(.p-datatable.p-datatable-hoverable-rows
    .p-datatable-tbody
    > tr:not(.p-highlight):hover) {
  background: rgba(119, 123, 241, 0.1);
}

::v-deep(.p-column-filter-menu-button:hover) {
  color: #2170e7;
}

::v-deep(.p-datatable .p-datatable-tbody > tr) {
  transition: background-color 0.2s, color 0.2s, border-color 0.2s,
    box-shadow 0.2s;
}

::v-deep(.p-button) {
  background: #464df2;
  color: #ffffff;
  border: 1px solid #464df2;
  margin: 0;
  outline: 0 none;
  border-radius: 6px;
  transition: background-color 0.2s, color 0.2s, border-color 0.2s,
    box-shadow 0.2s;
  cursor: pointer;
}

::v-deep(.p-button.p-button-info, .p-buttonset.p-button-info
    > .p-button, .p-splitbutton.p-button-info > .p-button) {
  color: #ffffff;
  background: #7fb4fa;
  border: 1px solid #7fb4fa;
}

::v-deep(.p-paginator .p-paginator-pages .p-paginator-page.p-highlight) {
  background: rgba(70, 77, 242, 0.9);
  color: #ffffff;
}

::v-deep(.p-paginator .p-paginator-pages .p-paginator-page) {
  padding: 0;
  border: 1px solid transparent;
  text-align: center;
  line-height: 2.286em;
  min-width: 2.286em;
  height: 2.286em;
  color: #83888f;
  margin: 0 0.125em;
  border-radius: 6px;
  transition: background-color 0.2s, color 0.2s, border-color 0.2s,
    box-shadow 0.2s;
}

::v-deep(.p-dialog .p-dialog-header) {
  border-top-right-radius: 24px;
  border-top-left-radius: 24px;
}

::v-deep(.p-dialog) {
  border-radius: 24px;
}

::v-deep(.p-dialog .p-dialog-header .p-dialog-title) {
  color: #69707a;
  margin-left: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

::v-deep(.p-dialog .p-dialog-footer) {
  border-bottom-right-radius: 24px;
  border-bottom-left-radius: 24px;
}

::v-deep(.p-fluid .p-inputtext) {
  border-radius: 8px;
}

::v-deep(.p-field > label) {
  font-weight: 500;
}

::v-deep(.p-dialog .p-dialog-content) {
  padding: 0 1.5rem 0rem 1.5rem;
}

::v-deep(.p-datatable .p-paginator-bottom) {
  border-width: 0 0 0 0;
}
.main-layout-details {
  border-radius: 24px;
  padding: 20px;
  background-color: white;
  box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 12px;
}
.active .main-layout-details {
  margin-left: 266.5px;
  transition: all 0.2s;
}
.inactive .main-layout-details {
  margin-left: 0;
  transition: all 0.2s;
}
</style>
