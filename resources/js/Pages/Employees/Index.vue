<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import DangerButton from '@/Components/DangerButton.vue';
import InputError from '@/Components/InputError.vue';
import InputLabel from '@/Components/InputLabel.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import SecondaryButton from '@/Components/SecondaryButton.vue';
import TextInput from '@/Components/TextInput.vue';
import SelectInput from '@/Components/SelectInput.vue';
import WarningButton from '@/Components/WarningButton.vue';
import Modal from '@/Components/Modal.vue';
import { Head, Link, useForm } from '@inertiajs/vue3';
import { nextTick, ref } from 'vue';
import Swal from 'sweetalert2';
import VueTailwindPagination from '@ocrv/vue-tailwind-pagination';

const nameInput = ref(null);
const modal = ref(false);
const title = ref('');
const operation = ref(1);
const id = ref('');

const props = defineProps({
    employees: {type:Object},
    departments: {type: Object}
});

const form = useForm({
    name:'', email:'', phone:'', department_id:''
});
const formPage = useForm({});
const onPageClick = (event) => {
    formPage.get(route('employees.index', {page:event}));
}
const openModal = (op, name, email, phone, department, employee) =>{
    modal.value =true;
    nextTick = ( () => nameInput.value.focus());
    operation.value = op;
    id.value = employee;
    if (op == 1){
        title.value = 'Create employee';
    }
    else{
        title.value = 'Edit employee';
        form.name = name;
        form.email = email;
        form.phone = phone;
        form.department_id = department;
    }
}
const closeModal = () =>{
    modal.value = false;
    form.reset();
}
const save = () =>{
    if (operation.value ==1) {
        form.post(route('employees.store'), {
            onSuccess: ()=>{ok('Employee created')}
        });
        
    } else {
        form.put(route('employees.update', id.value), {
            onSuccess: ()=>{ok('Empleado editado')}
        });
    }
}
const ok = (msj) => {
    form.reset();
    closeModal();
    Swal.fire({title:msj, icon:'success'});
}
const deleteEmployee = (id, name) => {
    const alerta = Swal.mixin({
        buttonStyling:true
    });
    alerta.fire({
        title: 'Seguro que desea eliminar '+name+' ?',
        icon:'question', showCancelButton:true,
        confirmButtonText: '<i class="fa-solid fa-check"></i> Sí',
        cancelButtonText: '<i class="fa-solid fa-ban"></i> Cancelar',
    }).then((result) => {
        if(result.isConfirmed) {
            form.delete(route('employees.destroy', id), {
                onSuccess: () => {ok('Empleado eliminado')}
            });
        }
    });
}
</script>

<template>
    <Head title="Empleados" />

    <AuthenticatedLayout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">Departamentos</h2>
        </template>

        <div class="py-12">
            <div class="bg-white grid v-screen place-items-center">
                <div class="mt-3 mb-3 flex">
                    <!-- Contenido -->
                    <PrimaryButton @click="$event => openModal(1)">
                        <i class="fa-solid fa-plus-circle"></i> Add
                    </PrimaryButton>
                </div>
                <div class="bg-white grid v-screen place-items-center">
                    <table class="table-auto border border-gray-400" >
                        <thead>
                            <tr class="bg-gray-100">
                                <th class="px-4 py-4">#</th>
                                <th class="px-4 py-4">Nombre</th>
                                <th class="px-4 py-4">Email</th>
                                <th class="px-4 py-4">Telefono</th>
                                <th class="px-4 py-4">Departamento</th>
                                <th class="px-4 py-4">Nombre</th>
                                <th class="px-4 py-4"></th>
                                <th class="px-4 py-4"></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="em, i in employees.data" :key="em.id">
                                <td class="border border-gray-400 px-2 py-2">{{ i+1 }}</td>
                                <td class="border border-gray-400 px-2 py-2">{{ em.name }}</td>
                                <td class="border border-gray-400 px-2 py-2">{{ em.email }}</td>
                                <td class="border border-gray-400 px-2 py-2">{{ em.phone }}</td>
                                <td class="border border-gray-400 px-2 py-2">{{ em.department }}</td>
                                <td class="border border-gray-400 px-2 py-2">
                                    <WarningButton @click="$event=>openModal(2, em.name, em.email, em.phone, em.department_id, em.id)">
                                        <i class="fa-solid fa-edit"></i> Editar
                                    </WarningButton>
                                </td>
                                <td class="border border-gray-400 px-2 py-2">
                                    <DangerButton @click="deleteEmployee(em.id)">
                                        <i class="fa-solid fa-trash"></i>
                                    </DangerButton>
                                </td>
                            </tr> 
                        </tbody>
                    </table>
                </div>
                <div class="bg-white grid v-screen place-items-center">
                    <VueTailwindPagination
                        :current="employees.currentPage" :total="employees.total"
                        :per-page="employees.perPage" 
                        @page-changed="$event => onPageClick($event)" >
                    </VueTailwindPagination>
                </div>
            </div>
        </div>
    </AuthenticatedLayout>
</template>
