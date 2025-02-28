<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";
	import { on } from "svelte/events";

    let data = [];
    let isLoading = true;
    let expandedRow = null;
    let showDownloadMessage = false;
    let showUpdateModal = false;
    let selectedRow = null;
    let customers = [];
    let bsNumbers = [];
    let units = [];
    let poStatuses = [];
    let concernsOnOrders = [];
    let filteredData = [];
    let searchQuery = "";

    let UpdateData = {
        sra_engineer_name: "",
        supplier: "",
        customer_name: "",
        bs_no: "",
        customer_po_no: "",
        po_date: "",
        part_code: "",
        quantity: "",
        unit: "",
        total_value: "",
        po_status_dd: "",
        concerns_on_order: "",
        delivery_schedule_as_per_customer_po: "",
        customer_clearance_for_billing: "",
        reserved_quantity_from_stock: "",
        mei_po_no: "",
        month_of_delivery_scheduled: "",
        category: "",
        so_number: ""

    };

    let isUpdating = false;
    const fetchurl = "https://srcustomerpobackend.onrender.com/fetch";
    const downloadexcelurl = "https://srcustomerpobackend.onrender.com/download";
    const updateurl = "https://srcustomerpobackend.onrender.com/update";

    async function fetchDropdownData() {
        try {
            const response = await fetch("https://srcustomerpobackend.onrender.com/dropdown");
            if (response.ok) {
                const dropdownData = await response.json();
                customers = [...new Set(dropdownData.map(item => item.customer_name))];
                bsNumbers = [...new Set(dropdownData.map(item => item.bs_no))];
                units = [...new Set(dropdownData.map(item => item.unit))];
                poStatuses = [...new Set(dropdownData.map(item => item.po_status_dd))];
                concernsOnOrders = [...new Set(dropdownData.map(item => item.concerns_on_order))];
            } else {
                console.error("Error fetching dropdown data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching dropdown data:", error);
        }
    }

    async function fetchData() {
        try {
            const response = await fetch(fetchurl);
            if (response.ok) {
                data = await response.json();
                isLoading = false;
                filteredData = [...data];
            } else {
                console.error("Error fetching data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        }
    }

    onMount(() => {
        fetchDropdownData();
        fetchData();
    });

    function openUpdateModal(row) {
        UpdateData = {
            sra_engineer_name: row.sra_engineer_name || "",
            supplier: row.supplier || "",
            customer_name: row.customer_name || "",
            bs_no: row.bs_no || "",
            customer_po_no: row.customer_po_no || "",
            po_date: row.po_date || "",
            part_code: row.part_code || "",
            quantity: row.quantity || "",
            unit: row.unit || "",
            total_value: row.total_value || "",
            po_status_dd: row.po_status_dd || "",
            concerns_on_order: row.concerns_on_order || "",
            delivery_schedule_as_per_customer_po: row.delivery_schedule_as_per_customer_po || "",
            customer_clearance_for_billing: row.customer_clearance_for_billing || "",
            reserved_quantity_from_stock: row.reserved_quantity_from_stock || "",
            mei_po_no: row.mei_po_no || "",
            month_of_delivery_scheduled: row.month_of_delivery_scheduled || "",
            category: row.category || "",
            so_number: row.so_number || ""
        };

        selectedRow = row;
        showUpdateModal = true;
    }

    async function updateCustomer(event) {
        event.preventDefault();
        isUpdating = true;

        try {
            const updatedFields = {};
            Object.keys(UpdateData).forEach((key) => {
                if (UpdateData[key] !== selectedRow[key]) {
                    updatedFields[key] = UpdateData[key];
                }
            });

            if (Object.keys(updatedFields).length === 0) {
                console.log("No fields updated.");
                isUpdating = false;
                return;
            }

            const updatedData = { ...selectedRow, ...updatedFields };

            const response = await fetch(updateurl, {
                method: "POST",
                body: JSON.stringify(updatedData),
            });

            if (response.ok) {
                console.log("Customer PO updated successfully.");
                showUpdateModal = false;
                selectedRow = null;
                await fetchData(); 
                console.log(data)
            } else {
                console.error("Failed to update customer PO:", response.statusText);
            }
        } catch (error) {
            console.error("Error updating customer PO:", error);
        } finally {
            isUpdating = false;
        }
    }

    async function downloadExcelCPO() {
        try {
            const response = await fetch(downloadexcelurl);
            if (response.ok) {
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = "customerpo.xlsx";
                a.click();
                window.URL.revokeObjectURL(url);
            } else {
                console.error("Error downloading Excel:", response.statusText);
            }
        } catch (error) {
            console.error("Error downloading Excel:", error);
        }
    }

    function convertToIST(timestamp) {
        if (!timestamp) return "Invalid Date";
        const date = new Date(timestamp);
        if (isNaN(date.getTime())) return "Invalid Date";

        const options = {
            timeZone: "Asia/Kolkata",
            year: "numeric",
            month: "numeric",
            day: "numeric",
            hour: "2-digit",
            minute: "2-digit",
            second: "2-digit",
            hour12: true,
        };

        return new Intl.DateTimeFormat("en-IN", options)
            .format(date)
            .replace(/(AM|PM)/g, (match) => ` ${match}`);
    }
    function searchTable() {
    const query = searchQuery.toLowerCase().trim();
    if (!query) {
        filteredData = [...data]; 
        return;
    }

    filteredData = data.filter(row => {
        const engineerName = row.sra_engineer_name?.toLowerCase() || ""; 
        const customerName = row.customer_name?.toLowerCase() || ""; 
        return engineerName.includes(query) || customerName.includes(query);
    });
}
function formatToInputDate(timestamp) {
  if (!timestamp) return "";
  const date = new Date(timestamp);
  return date.toISOString().split("T")[0]; // Returns "YYYY-MM-DD"
}
function formatDateToDayMonthYear(dateString) {
    if (!dateString) return "N/A";
    const date = new Date(dateString);
    return date.toLocaleDateString("en-GB", {
      day: "numeric",
      month: "long",
      year: "numeric"
    });
  }

</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />

    <main class="flex-grow py-24 px-2">
        <div class="relative w-full flex justify-end bg-white">
            <input
              type="text"
              bind:value={searchQuery}
              on:input={searchTable}
              placeholder="Search by Engineer or Customer Name"
              class="px-3 py-1  w-80 mb-4 shadow-md border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none "
            />
          </div>
                  {#if isLoading}
            <div class="flex justify-center items-center h-full">
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800"></div>
            </div>
        {:else}
            <div class="overflow-x-auto bg-gray-100 shadow-lg rounded-lg">
                <table class="w-full border-collapse text-sm text-black">
                    <thead class="bg-black text-white">
                        <tr>
                            <th class="py-3 px-4">ID</th>

                            <th class="py-3 px-4">SRA Engineer Name</th>
                            <th class="py-3 px-4">Supplier</th>
                            <th class="py-3 px-4">Customer Name</th>
                            <th class="py-3 px-4">BS No</th>
                            <th class="py-3 px-4">Customer PO No</th>
                            <th class="py-3 px-4">PO Date</th>
                            <th class="py-3 px-4">Part Code</th>
                            <th class="py-3 px-4">Quantity</th>
                            <th class="py-3 px-4">Unit</th>
                            <th class="py-3 px-4">Total Value</th>
                            <th class="py-3 px-4">PO Status</th>
                            <th class="py-3 px-4">Concerns</th>
                            <th class="py-3 px-4">Billable Value</th>
                            <th class="py-3 px-4">Delivery Schedule</th>
                            <th class="py-3 px-4">Customer Clearance</th>
                            <th class="py-3 px-4">Reserved Quantity</th>
                            <th class="py-3 px-4">Required Quantity</th>
                            <th class="py-3 px-4">Pending Quantity</th>
                            <th class="py-3 px-4">Material Due Quantity</th>
                            <th class="py-3 px-4">SO Number</th>
                            <th class="py-3 px-4">MEI PO No</th>
                            <th class="py-3 px-4">Final PO Status</th>
                            <th class="py-3 px-4">Pending Value</th>
                            <th class="py-3 px-4">Pending Order Value</th>
                            <th class="py-3 px-4">Reserved Stock Value</th>
                            <th class="py-3 px-4">Month of Delivery</th>
                            <th class="py-3 px-4">Category</th>
                            <th class="py-3 px-4">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#if data.length === 0}
                            <tr>
                                <td colspan="30" class="py-4 text-center text-gray-600">No data available</td>
                            </tr>
                        {:else}
                            {#each filteredData as row, index}
                            <tr class="border-t border-gray-300 hover:bg-gray-200">
                                <td class="py-3 px-4 text-center">{row.id}</td>

                                <td class="py-3 px-4 text-center">{row.sra_engineer_name}</td>
                                <td class="py-3 px-4 text-center">{row.supplier}</td>
                                <td class="py-3 px-4 text-center">{row.customer_name}</td>
                                <td class="py-3 px-4 text-center">{row.bs_no}</td>
                                <td class="py-3 px-4 text-center">{row.customer_po_no}</td>
                                <td class="py-3 px-4 text-center">{formatDateToDayMonthYear(row.po_date) ?? "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.part_code}</td>
                                <td class="py-3 px-4 text-center">{row.quantity}</td>
                                <td class="py-3 px-4 text-center">{row.unit}</td>
                                <td class="py-3 px-4 text-center">₹{row.total_value ? row.total_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">{row.po_status_dd}</td>
                                <td class="py-3 px-4 text-center">{row.concerns_on_order}</td>
                                <td class="py-3 px-4 text-center">₹{row.billable_scheduled_value ? row.billable_scheduled_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">{formatDateToDayMonthYear(row.delivery_schedule_as_per_customer_po) ?? "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.customer_clearance_for_billing ?? "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.reserved_quantity_from_stock ?? 0}</td>
                                <td class="py-3 px-4 text-center">{row.required_quantity_to_order ?? 0}</td>
                                <td class="py-3 px-4 text-center">{row.pending_quantity_against_po ?? 0}</td>
                                <td class="py-3 px-4 text-center">{row.material_due_quantity ?? 0}</td>
                                <td class="py-3 px-4 text-center">{row.so_number}</td>
                                <td class="py-3 px-4 text-center">{row.mei_po_no}</td>
                                <td class="py-3 px-4 text-center">{row.po_status_final}</td>
                                <td class="py-3 px-4 text-center">₹{row.pending_value_against_po ? row.pending_value_against_po.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">₹{row.pending_order_value ? row.pending_order_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">₹{row.reserved_quantity_stock_value ? row.reserved_quantity_stock_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">{formatDateToDayMonthYear(row.month_of_delivery_scheduled) ?? "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.category}</td>
                        
                                <td class="py-3 px-4 text-center">
                                    <!-- svelte-ignore a11y_consider_explicit_label -->
                                    <button
                                    class="text-xl font-medium rounded-full flex items-center justify-center text-center"
                                    on:click={() => openUpdateModal(row)}
                                >
                                    <i class="fas fa-edit"></i>
                                </button>
                                       
                                    </td>
                                </tr>
                            {/each}
                        {/if}
                    </tbody>
                </table>
            </div>
            <div class="py-4 flex justify-center items-center fixed bottom-0 left-0 right-0 text-center">
                <button class="text-white bg-black rounded-md px-9 py-2" 
                on:click={downloadExcelCPO}
                >
                    Download
                </button>
            </div>
        {/if}
    </main>
    {#if showDownloadMessage}
    <div class="fixed bottom-12 right-4 bg-black text-white font-semibold p-4 rounded-md shadow-2xl duration-300" transition:fade>
        Excel downloaded successfully!
    </div>
{/if}
  

{#if showUpdateModal}
<div class="fixed inset-0 bg-black opacity-50 z-40"></div> <!-- Background Overlay -->
<div class="fixed inset-0 flex items-center justify-center z-50">
    <div class="w-full max-w-4xl mx-4 bg-white rounded-lg p-4 shadow-lg relative">

        <form on:submit={updateCustomer} class="bg-white shadow-xl rounded-lg p-8 space-y-8">
            <h1 class="text-center text-xl py-2 mb-6 font-semibold text-gray-900">Update Customer PO </h1>

        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">


                <div>
                    <label for="sra_engineer_name" class="block text-sm font-medium text-gray-700">SRA Engineer Name</label>
                    <input type="text" 
                        id="sra_engineer_name"
                        bind:value={UpdateData.sra_engineer_name}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                    <input type="text" 
                        id="supplier"
                        bind:value={UpdateData.supplier}
                        placeholder="Enter Supplier" 
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="customer" class="block text-sm font-medium text-gray-700">Customer Name</label>
                    <select
                        id="customer"
                        bind:value={UpdateData.customer_name}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a Customer</option>
                        {#each customers as customer}
                            <option value={customer}>{customer}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="bs_no" class="block text-sm font-medium text-gray-700">BS Number</label>
                    <select
                        id="bs_no"
                        bind:value={UpdateData.bs_no}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a BS No</option>
                        {#each bsNumbers as bsNumber}
                            <option value={bsNumber}>{bsNumber}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="customer_po_no" class="block text-sm font-medium text-gray-700">Customer P.O. No</label>
                    <input type="text" 
                        id="customer_po_no"
                        bind:value={UpdateData.customer_po_no}
                        placeholder="Enter Customer PO Number" 
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="po_date" class="block text-sm font-medium text-gray-700">PO Date</label>
                    <input type="date" 
                        id="po_date"
                        bind:value={UpdateData.po_date}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="part_code" class="block text-sm font-medium text-gray-700">Part Code</label>
                    <input
                        placeholder="Enter Part Code"
                        id="part_code"
                        type="text"
                        bind:value={UpdateData.part_code}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="quantity" class="block text-sm font-medium text-gray-700">Quantity</label>
                    <input
                        placeholder="Enter Quantity"
                        id="quantity"
                        type="number"
                        bind:value={UpdateData.quantity}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                    <select
                        id="unit"
                        bind:value={UpdateData.unit}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a Unit</option>
                        {#each units as unit}
                            <option value={unit}>{unit}</option>
                        {/each}
                    </select>
                </div>
                <div>
                    <label for="total_value" class="block text-sm font-medium text-gray-700">Total Value</label>
                    <input type="number"
                        id="total_value"
                        bind:value={UpdateData.total_value}
                        placeholder="Enter Total Value"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="po_status_dd" class="block text-sm font-medium text-gray-700">PO Status</label>
                    <select
                        id="po_status_dd"
                        bind:value={UpdateData.po_status_dd}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select PO Status</option>
                        {#each poStatuses as status}
                            <option value={status}>{status}</option>
                        {/each}
                    </select>
                </div>
                <div>
                    <label for="concerns_on_order" class="block text-sm font-medium text-gray-700">Concerns on Order</label>
                    <select
                        id="concerns_on_order"
                        bind:value={UpdateData.concerns_on_order}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    >
                        <option value="" disabled>Select a Concern</option>
                        {#each concernsOnOrders as concern}
                            <option value={concern}>{concern}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="delivery_schedule_as_per_customer_po" class="block text-sm font-medium text-gray-700">
                        Delivery Schedule - As per Customer PO
                    </label>
                    <input type="date" 
                        id="delivery_schedule_as_per_customer_po"
                        bind:value={UpdateData.delivery_schedule_as_per_customer_po}
                        placeholder="Enter Delivery Schedule"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="customer_clearance_for_billing" class="block text-sm font-medium text-gray-700">
                        Customer Clearance for Billing
                    </label>
                    <input type="text" 
                        id="customer_clearance_for_billing"
                        bind:value={UpdateData.customer_clearance_for_billing}
                        placeholder="Enter Clearance Status"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="reserved_quantity_from_stock" class="block text-sm font-medium text-gray-700">
                        Reserved Quantity from Stock
                    </label>
                    <input type="number" 
                        id="reserved_quantity_from_stock"
                        bind:value={UpdateData.reserved_quantity_from_stock}
                        placeholder="Enter Reserved Quantity"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="so_number" class="block text-sm font-medium text-gray-700">
                        SO Number
                    </label>
                    <input type="text" 
                        id="so_number"
                        bind:value={UpdateData.so_number}
                        placeholder="Enter SO Number"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                    <div>
                        <label for="mei_po_no" class="block text-sm font-medium text-gray-700">
                            MEI PO No
                        </label>
                        <input type="text" 
                            id="mei_po_no"
                            bind:value={UpdateData.mei_po_no}
                            placeholder="Enter MEI PO No"
                            class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                            required
                        />
                    </div>

         
            <div>
                <label for="month_of_delivery_scheduled" class="block text-sm font-medium text-gray-700">
                    Month of Delivery Scheduled
                </label>
                <input type="date" 
                    id="month_of_delivery_scheduled"
                    bind:value={UpdateData.month_of_delivery_scheduled}
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required
                />
            </div>

     
            <div>
                <label for="category" class="block text-sm font-medium text-gray-700">
                    Category
                </label>
                <input type="text" 
                    id="category"
                    bind:value={UpdateData.category}
                    placeholder="Enter Category"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required
                />
            </div>           
        </div>
            <div class="flex justify-end">
                <button type="submit" class="px-6 py-2 bg-black text-white rounded-md" disabled={isUpdating}>
                
                    {#if isUpdating}
                        Updating...
                    {:else}
                        Update 
                    {/if}
                </button>
            </div>
        
       
        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
            class="absolute top-4 right-4 text-gray-500"
            on:click={() => showUpdateModal = false}
        >
            <i class="fas fa-times"></i>
        </button>
        </form>
    </div>
</div>

{/if}
</div>



