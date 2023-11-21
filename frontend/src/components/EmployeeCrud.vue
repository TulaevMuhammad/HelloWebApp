<template>
  <div class="row">
    <div class="row">
      <button @click="showCreateCompanyForm" class="btn btn-success mb-2">Create Company</button>
    </div>

    <div class="row">
      <button @click="showCreateDepartmentForm" class="btn btn-info mb-2">Create Department</button>
    </div>

    <form class="mt-3 d-flex">
      <label class="mr-2 mb-2">
        Company:
        <select v-model="selectedCompanyId" @change="submitInfo" class="form-select">
          <option v-for="company in companyOptions" :key="company.id" :value="company.id">
            {{ company.name }}
          </option>
        </select>
      </label>

      <label class="mr-2 mb-2">
        Department:
        <select v-model="selectedDepartmentId" @change="submitInfo" class="form-select">
          <option v-for="department in departmentOptionsByCompany" :key="department.id" :value="department.id">
            {{ department.name }}
          </option>
        </select>
      </label>
    </form>

    <div class="row">
      <button @click="showCreateForm" class="btn btn-success mb-2">Create Employee</button>
    </div>

    <div v-if="isCreateCompanyFormVisible" class="mt-3">
      <form @submit.prevent="createCompany">
        <label class="mr-2">
          Company Name:
          <input v-model="newCompanyName" class="form-control" />
        </label>
        <button type="submit" class="btn btn-primary">Create</button>
      </form>
    </div>

    <div v-if="isCreateDepartmentFormVisible" class="mt-3">
      <form @submit.prevent="createDepartment">
        <label class="mr-2">
          Company:
          <select v-model="selectedCompanyForDepartment" class="form-select">
            <option v-for="company in companyOptions" :key="company.id" :value="company.id">
              {{ company.name }}
            </option>
          </select>
        </label>
        <label class="mr-2">
          Department Name:
          <input v-model="newDepartmentName" class="form-control" />
        </label>
        <button type="submit" class="btn btn-primary">Create</button>
      </form>
    </div>

    <div v-if="isCreateFormVisible" class="mt-3">
      <form @submit.prevent="createEmployee">
        <label class="mr-2">
          Employee Name:
          <input v-model="newEmployeeName" class="form-control" />
        </label>
        <button type="submit" class="btn btn-primary">Create</button>
      </form>
    </div>
  </div>


  <div class="row">
    <form @submit.prevent="submitInfo" class="mt-3 d-flex">




      <label class="mr-2 mb-2">
        Page:
        <input type="number" v-model.number="page" class="form-control" />
      </label>
      <label class="mr-2 mb-2">
        Size:
        <input type="number" v-model.number="size" class="form-control" />
      </label>
      <label class="mr-2 mb-2">
        Sort:
        <select v-model="sort" class="form-select">
          <option value="employee_id">Employee ID</option>
          <option value="employee_name">Employee Name</option>
          <option value="department_name">Department Name</option>
          <option value="company_name">Company Name</option>
        </select>

      </label>

      <label class="mr-2 mb-2">
        Company:
        <select v-model="selectedCompanyId" @change="submitInfo" class="form-select">
          <option v-for="company in companyOptions" :key="company.id" :value="company.id">
            {{ company.name }}
          </option>
        </select>
      </label>

      <label class="mr-2 mb-2">
        Department:
        <select v-model="selectedDepartmentId2" class="form-select">
          <option v-for="department in departmentOptionsByCompany" :key="department.id" :value="department.id">
            {{ department.name }}
          </option>
        </select>
      </label>

      <button @click="sendRequest" type="submit" class="btn btn-primary mb-2">Send</button>
    </form>
  </div>

  <div class="container">
    <div class="row">
      <div class="col-md-8">
        <table class="table mt-3">
          <thead>
            <tr>
              <th>Employee ID</th>
              <th>Employee Name</th>
              <th>Department Name</th>
              <th>Company Name</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="employee in data" :key="employee.employeeId">
              <td>{{ employee.employeeId }}</td>
              <td>{{ employee.employeeName }}</td>
              <td>{{ employee.departmentName }}</td>
              <td>{{ employee.companyName }}</td>
              <td>
            <tr>
              <button @click="showUpdateForm(employee)" class="btn btn-primary">Update</button>
              <button @click="deleteEmployee(employee.employeeId)" class="btn btn-danger">Delete</button>
            </tr>
            </td>
            </tr>
          </tbody>
        </table>

        <div v-if="isUpdateFormVisible" class="mt-3">
          <form @submit.prevent="updateEmployee">
            <label class="mr-2">
              New Name:
              <input v-model="updatedName" class="form-control" />
            </label>
            <label class="mr-2">
              New Department:
              <select v-model="updatedDepartmentId" class="form-select">
                <option v-for="department in updatedDepartmentOptions" :key="department.id" :value="department.id">
                  {{ department.name }}
                </option>
              </select>
            </label>
            <button type="submit" class="btn btn-primary">Update</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';


const companyOptions = ref([]); // Declare it at the top level
const updatedDepartmentOptions = ref([]); // Declare it at the top level
const page = ref(1);
const size = ref(50);
const sort = ref('employee_id');
const data = ref([]);
const isUpdateFormVisible = ref(false);
const updatedEmployee = ref(null);
const updatedName = ref('');
const updatedDepartmentId = ref(null);
const selectedCompanyId = ref(null);
const selectedDepartmentId = ref(null)
const selectedDepartmentId2 = ref(null)


const departmentOptions = ref([]);
const departmentOptionsByCompany = ref([]);
const isCreateFormVisible = ref(false);
const newEmployeeName = ref('');
const isCreateCompanyFormVisible = ref(false);
const newCompanyName = ref('');
const isCreateDepartmentFormVisible = ref(false);
const selectedCompanyForDepartment = ref(null);
const newDepartmentName = ref('');

const showCreateCompanyForm = () => {
  // Reset the newCompanyName when showing the create company form
  newCompanyName.value = '';
  isCreateDepartmentFormVisible.value = false;

  isCreateCompanyFormVisible.value = true;
};

const showCreateDepartmentForm = () => {
  isCreateCompanyFormVisible.value = false;

  selectedCompanyForDepartment.value = null;
  newDepartmentName.value = '';
  isCreateDepartmentFormVisible.value = true;
};

const createDepartment = async () => {
  // Check if a company is selected
  if (!selectedCompanyForDepartment.value || selectedCompanyForDepartment.value <= 0) {
    alert('Please select a company before creating a department.');
    return;
  }
  const newDepartmentData = {
    name: newDepartmentName.value,
    companyId: selectedCompanyForDepartment.value,
  };

  try {
    await axios.post('http://localhost:8081/employeeResource/saveDepartment', newDepartmentData);
    // Update the data after successful creation
    submitInfo();
    // Close the create department form
    isCreateDepartmentFormVisible.value = false;
  } catch (error) {
    console.error('Error creating department:', error);
  }
};


const createCompany = async () => {
  const newCompanyData = {
    name: newCompanyName.value,
  };

  try {
    await axios.post('http://localhost:8081/employeeResource/saveCompany', newCompanyData);
    // Update the data after successful creation
    submitInfo();
    // Close the create company form
    isCreateCompanyFormVisible.value = false;
  } catch (error) {
    console.error('Error creating company:', error);
  }
};


const baseURL = 'http://localhost:8081/employeeResource/';

const submitInfo = async () => {

  try {

    let url;

    console.log("selectedDepartmentId2.value:", selectedDepartmentId2.value);

    console.log("selectedCompanyId.value:", selectedCompanyId.value);


    if (selectedDepartmentId2.value > 0) {
      // If a department is selected and its ID is not 0
      url = `getEmployeesByDepartmentId/${selectedDepartmentId2.value}?page=${page.value}&size=${size.value}&sort=${sort.value}`;
      selectedDepartmentId2.value=0;
    } else {
      // If no department is selected or its ID is 0
      url = `getAllEmployees?page=${page.value}&size=${size.value}&sort=${sort.value}`;
      selectedDepartmentId2.value=0;
    }
    
    const employeeResponse = await axios.get(
      url,
      { baseURL }
    );

    data.value = employeeResponse.data;

    const departmentResponse = await axios.get('getAllDepartments', { baseURL });
    departmentOptions.value = departmentResponse.data;

    const companyResponse = await axios.get('getAllCompanies', { baseURL });
    companyOptions.value = companyResponse.data;

    // Fetch departments based on the selected company


    if (selectedCompanyId.value) {
      const departmentByCompanyResponse = await axios.get(
        `getDepartmentByCompany/${selectedCompanyId.value}`,
        { baseURL: 'http://localhost:8081/employeeResource/' }
      );
      departmentOptionsByCompany.value = departmentByCompanyResponse.data;
    }

    
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};




const showUpdateForm = async (employee) => {
  updatedEmployee.value = employee;
  updatedName.value = employee.employeeName;
  updatedDepartmentId.value = employee.departmentId;

  // Fetch the updated department options based on the selected company
  try {
    if (employee.companyId) {
      const response = await axios.get(`http://localhost:8081/employeeResource/getDepartmentByCompany/${employee.companyId}`);
      updatedDepartmentOptions.value = response.data;
    } else {
      // If no company is associated with the employee, fetch all departments
      const response = await axios.get('http://localhost:8081/employeeResource/getAllDepartments');
      updatedDepartmentOptions.value = response.data;
    }
  } catch (error) {
    console.error('Error fetching updated department options:', error);
  }

  isUpdateFormVisible.value = true;
};

const updateEmployee = async () => {
  const updatedEmployeeData = {
    id: updatedEmployee.value.employeeId,
    name: updatedName.value,
    departmentId: updatedDepartmentId.value,
  };

  try {
    await axios.put('http://localhost:8081/employeeResource/updateEmployee', updatedEmployeeData);
    // Update the data after successful update
    submitInfo();
    // Close the update form
    isUpdateFormVisible.value = false;
  } catch (error) {
    console.error('Error updating employee:', error);
  }
};

const showCreateForm = () => {
  // Reset the newEmployeeName when showing the create form
  newEmployeeName.value = '';
  isCreateFormVisible.value = true;
};

const createEmployee = async () => {
  // Check if a department is selected
  if (!selectedDepartmentId.value || selectedDepartmentId.value <= 0) {
    alert('Please select a department before creating an employee.');
    return;
  }

  const newEmployeeData = {
    name: newEmployeeName.value,
    departmentId: selectedDepartmentId.value,
  };

  try {
    await axios.post('http://localhost:8081/employeeResource/saveEmployee', newEmployeeData);
    // Update the data after successful creation
    submitInfo();
    // Close the create form
    isCreateFormVisible.value = false;
  } catch (error) {
    console.error('Error creating employee:', error);
  }
};

const deleteEmployee = async (id) => {
  try {
    await axios.delete(`http://localhost:8081/employeeResource/deleteEmployee/${id}`);
    // Update the data after successful deletion
    submitInfo();
  } catch (error) {
    console.error('Error deleting employee:', error);
  }
};


onMounted(async () => {
  const companyResponse = await axios.get('http://localhost:8081/employeeResource/getAllCompanies');
  departmentOptions.value = companyResponse.data;
});

onMounted(async () => {
  const response = await axios.get('http://localhost:8081/employeeResource/getAllDepartments');
  updatedDepartmentOptions.value = response.data; // Use updatedDepartmentOptions here
});

onMounted(() => {
  submitInfo();
});




</script>


<style>
body {
  background: linear-gradient(to right, #fcfbfc, #13e60c);
  /* Background gradient */
}

/* Add animation to the table */
table {
  animation: fadeIn 1s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

/* Add border to the form elements */
label,
input,
select,
button {
  border: 1px solid #333;
  border-radius: 5px;
  padding: 8px;
}

/* Change text color for labels and buttons */
label,
input,
select,
button,
th,
td {
  color: #333;
}

/* Apply filters to table cells */
th,
td {
  filter: grayscale(10%);
}

/* Apply layout and spacing to the form */
.row {
  display: flex;
  justify-content: space-between;
}

/* Add list-style to the table rows */
tbody tr {
  list-style: square;
}

/* Add miscellaneous styles to buttons */
.btn {
  cursor: pointer;
  transition: background-color 0.3s ease-in-out;
}

.btn:hover {
  background-color: #4CAF50;
  color: white;
}

/* Apply transform to the update form */
.update-form {
  transform: rotate(5deg);
}

/* Apply transition to the update form */
.update-form {
  transition: transform 0.5s ease-in-out;
}


body {
  background-color: white;
  /* Set your desired background color */
}

.row {
  margin-bottom: 20px;
  /* Add more space below the form */
}

form {
  margin-bottom: 20px;
  /* Add more space below the form inputs */
}

label,
input,
select,
button {
  margin-bottom: 10px;
  /* Add more space below these elements */
}


label,
input,
select,
button,
th,
td {
  color: black;
  /* Set your desired text color */
}

table {
  margin-top: 20px;
  /* Add more space above the table */
  margin-bottom: 20px;
  /* Add more space below the table */
}

th,
td {
  padding: 10px;
  /* Add padding to table cells for spacing */
}

.mt-3 {
  margin-top: 20px !important;
  /* Add more space using the utility class mt-3 */
}

.btn {
  margin-right: 10px;
  /* Add more space to the right of buttons */
}

.btn-primary {
  background-color: #3498db;
  /* Blue color for "Update" button */
  color: white;
}

.btn-danger {
  background-color: #e74c3c;
  /* Red color for "Delete" button */
  color: white;
}

.update-form {
  margin-top: 20px;
  /* Add more space above the update form */
  margin-bottom: 20px;
  /* Add more space below the update form */
}
</style>

