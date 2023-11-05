<script>
import Dialog from 'primevue/dialog';
import { useToast } from 'primevue/usetoast';
import { ref, onBeforeMount, onMounted, computed } from 'vue';
export default {
	data() {
		const toast = useToast();
		const visibleUpdate = ref(false);
		const visibleDelete = ref(false);
		const nomp = ref('');
		const cantidad = ref('');
		const precioUnitario = ref('');
		const subT = ref('');
		const ivaa = ref('');
		const totalConIvaa = ref('');

		async function pasoD(row) {
			subT.value = row.subTotal;
			ivaa.value = (row.subTotal * 0.16);
			totalConIvaa.value = ivaa.value + row.subTotal;

			this.productoItem.subTotal = subT.value;
			this.productoItem.iva = ivaa.value;
			this.productoItem.totalConIva = totalConIvaa.value;
		}

		async function editarProducto(row) {

			nomp.value = row.nomProducto;
			precioUnitario.value = row.precioUnitario;
			cantidad.value = row.cantidad;

			this.productoItem.cns = row.cns;
			this.visibleUpdate = true;
		}

		async function updateProducto() {
			const productoIndex = this.tablaCompras.findIndex(item => item.cns === this.productoItem.cns);
			if (productoIndex !== -1) {
				this.tablaCompras[productoIndex].nomProducto = this.nomp;
				this.tablaCompras[productoIndex].cantidad = this.cantidad;
				this.tablaCompras[productoIndex].precioUnitario = "$" + this.precioUnitario;
				this.tablaCompras[productoIndex].precioParcial = this.cantidad * this.precioUnitario;
			}
			this.visibleUpdate = false;
		}

		async function eliminarProducto(row) {
			this.visibleDelete = true;
		}

		async function closeU() {
			this.visibleUpdate = false;
		}

		async function closeD() {
			this.visibleDelete = false;
		}

		async function deleteProducto(row) {
			const index = this.tablaCompras.indexOf(row);
			if (index !== -1) {
				this.tablaCompras.splice(index, 1);
			}
			this.visibleDelete = false;
		}

		async function registrarC() {
			if (this.productoItem.nomProducto.trim() !== '' && this.productoItem.cantidad.trim() !== '' && this.productoItem.precioUnitario.trim() !== ''
			) {
				this.tablaCompras.push({
					cns: this.tablaCompras.length + 1,
					nomProducto: this.productoItem.nomProducto,
					cantidad: parseFloat(this.productoItem.cantidad),
					precioUnitario: parseFloat(this.productoItem.precioUnitario),
					precioParcial: parseFloat(this.productoItem.cantidad) * parseFloat(this.productoItem.precioUnitario),
				});

				toast.add({ severity: 'success', summary: 'Confirmación', detail: 'Compra Realizada Exitosamente!', life: 3000 });
				console.log(this.productoItem.precioParcial);
				this.productoItem.nomProducto = '';
				this.productoItem.cantidad = '';
				this.productoItem.precioUnitario = '';
			} else {
				toast.add({ severity: 'warn', summary: 'Alerta', detail: 'Registrar los datos solicitados!', life: 3000 });
			}
		}

		return {
			registrarC,
			toast,
			nomp,
			cantidad,
			precioUnitario,
			visibleDelete,
			visibleUpdate,
			editarProducto,
			deleteProducto,
			updateProducto,
			eliminarProducto,
			closeU,
			closeD,
			pasoD,
			tablaCompras: [
			],
			productoItem: {
				cns: null,
				nomProducto: null,
				cantidad: null,
				precioUnitario: null,
				precioParcial: null,
			},
			product: {
				subTotal: 0,
				iva: 0,
				totalConIva: 0,
			}
		}
	},
	created() {

	},
	methods: {
		formatoMoneda(value) {
			if (value)
				return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
			return;
		}
	},
	computed: {
		sub() {
			return this.tablaCompras.reduce((subtotal, producto) => {
				return subtotal + producto.precioParcial;
			}, 0);
		},
		iva() {
			return this.tablaCompras.reduce((iva, producto) => {
				return iva + (producto.precioParcial * 0.16);
			}, 0);
		},
		Total() {
			return this.tablaCompras.reduce((total, producto) => {
				return total + producto.precioParcial + (producto.precioParcial * 0.16);
			}, 0);
		}
	}
}
</script>


<template>
	<div class="p-grid">
		<Toast />
		<div class="p-col-12">
			<div class="card">
				<Panel header="PUNTO DE VENTA (PV)" style="height: 100%">
					<Toolbar class="p-mb-4">
						<template v-slot:start>
							<InputText type="text" size="40" placeholder="Nombre del producto"
								v-model="productoItem.nomProducto" />
							<InputText class="ml-8" type="text" placeholder="Piezas en existencia"
								v-model="productoItem.cantidad" />
							<InputText class="ml-8" type="text" placeholder="Precio Unitario $$"
								v-model="productoItem.precioUnitario" />
						</template>
						<template v-slot:end && #body="slotProps">
							<Button label="Registrar" icon="pi pi-plus" class="p-button-success p-mr-2"
								@click="registrarC()" />
						</template>
					</Toolbar>
					<br>

					<DataTable :value="tablaCompras" v-model:selection="productoItem" class="p-datatable-gridlines"
						dataKey="cns" :rows="5"
						paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
						:rowsPerPageOptions="[5, 10, 25]"
						currentPageReportTemplate="Visualizando {last} de {totalRecords} productos"
						style="margin-bottom: 20px" :paginator="true" responsiveLayout="scroll">

						<Column field="cns" header="Cns" :sortable="true" style="width:50px"></Column>
						<Column field="nomProducto" header="Nombre del Producto" style="width:370px"></Column>
						<Column field="precioUnitario" header="Precio Unitario" dataType="numeric" style="width:80px">
							<template #body="slotProps">
								{{ formatoMoneda(slotProps.data.precioUnitario) }}
							</template>
						</Column>
						<Column field="cantidad" header="Cantidad" style="width:60px;text-align:center"></Column>
						<Column field="precioParcial" header="Precio Final" style="width:80px">
							<template #body="slotProps">
								{{ formatoMoneda(slotProps.data.precioParcial) }}
							</template>
						</Column>
						<Column style="width:140px">
							<template #header>
								Acciones
							</template>
							<template #body="slotProps">
								<Button icon="pi pi-pencil" type="button" class="p-button-success p-mr-2 p-mb-1"
									@click="editarProducto(slotProps.data)"></Button>
								<Dialog v-model:visible="visibleUpdate" modal
									header="Actualización de los datos de un producto" :style="{ width: '45vw' }">
									<p>
									<div class="flex gap-2">
										<div class="flex-auto">
											<label for="nomp"><strong>Nombre del producto: </strong></label>
											<InputText class="ml-2" id="nomp" v-model="nomp" />
										</div>
										<div class="flex-auto">
											<label for="precioUnitario"><strong>Precio Unitario: </strong></label>
											<InputText class="ml-2" id="precioUnitario" v-model="precioUnitario" />
										</div>
										<div class="flex-auto">
											<label for="cantidad"><strong>Cantidad: </strong></label>
											<InputText class="ml-2" id="cantidad" v-model="cantidad" />
										</div>
									</div>
									<br>
									</p>
									<template #footer>
										<Button label="Actualizar" icon="pi pi-replay" @click="updateProducto()" />
										<Button label="Cancelar" icon="pi pi-cancel" @click="closeU()"></Button>
									</template>
								</Dialog>
								<Button icon="pi pi-trash" type="button" class="p-button-danger p-mb-1"
									@click="eliminarProducto(slotProps.data)"></Button>
								<Dialog v-model:visible="visibleDelete" modal
									header="¿En verdad desea elimnar el producto seleccionado?" :style="{ width: '30vw' }">
									<p>
										<br>
									</p>
									<template #footer>
										<Button label="Eliminar" severity="warning" icon="pi pi-check"
											@click="deleteProducto(slotProps.data)" autofocus />
										<Button label="Cancelar" icon="pi pi-cancel" @click="closeD()"></Button>
									</template>
								</Dialog>
							</template>
						</Column>
					</DataTable>

					<br>
					<Toolbar class="p-mb-4">
						<template v-slot:start>

						</template>
						<template v-slot:end>
							<label for="total">Subtotal: </label>
							<InputText class="ml-3" type="text" id="subT" placeholder="$ " :value="formatoMoneda(sub)"
								disabled />
							<label class="ml-5" for="total">IVA (16%): </label>
							<InputText class="ml-3" type="text" id="IVA" placeholder="$ " :value="formatoMoneda(iva)"
								disabled />
							<label class="ml-5" for="total">Total: </label>
							<InputText class="ml-3" type="text" id="totalCIva" placeholder="$ "
								:value="formatoMoneda(Total)" disabled />
						</template>
					</Toolbar>
				</Panel>
			</div>
		</div>
	</div>
</template>