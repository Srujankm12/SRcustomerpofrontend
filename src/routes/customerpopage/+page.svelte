<script>
    import { onMount } from 'svelte';
    import { writable, get } from 'svelte/store';
    import Header from '$lib/header.svelte';

    let formData = writable({
        sra_engineer_name: '',
        supplier: '',
        customer_name: '',
        bs_no: '',
        customer_po_no: '',
        po_date: '',
        part_code: '',
        quantity: '',
        unit: '',
        total_value: '',
        po_status_dd: '',
        concerns_on_order: '',
        delivery_schedule_as_per_customer_po: '',
        customer_clearance_for_billing: '',
        reserved_quantity_from_stock: '',
        mei_po_no: '',
        month_of_delivery_scheduled: '',
        category: '',
        so_number: ''
    });

    let customers = writable([]);
    let bsNumbers = writable([]);
    let units = writable([]);
    let poStatuses = writable([]);
    let concernsOnOrders = writable([]);

    const fetchDropdownData = async () => {
        try {
            const response = await fetch('https://srcustomerpobackend.onrender.com/dropdown');
            if (response.ok) {
                const data = await response.json();
                customers.set([...new Set(data.map(item => item.customer_name))]);
                bsNumbers.set([...new Set(data.map(item => item.bs_no))]);
                units.set([...new Set(data.map(item => item.unit))]);
                poStatuses.set([...new Set(data.map(item => item.po_status_dd))]);
                concernsOnOrders.set([...new Set(data.map(item => item.concerns_on_order))]);
            } else {
                console.error('Error fetching dropdown data:', response.statusText);
            }
        } catch (error) {
            console.error('Error fetching dropdown data:', error);
        }
    };

    onMount(fetchDropdownData);

  
    const handleSubmit = async (event) => {
        event.preventDefault();
        let formDataValue = get(formData);  

        try {
            const response = await fetch('https://srcustomerpobackend.onrender.com/submit', {
                method: 'POST', 
                body: JSON.stringify(formDataValue),
            });

            if (response.ok) {
                formData.set({
                    sra_engineer_name: '',
                    supplier: '',
                    customer_name: '',
                    bs_no: '',
                    customer_po_no: '',
                    po_date: '',
                    part_code: '',
                    quantity: '',
                    unit: '',
                    total_value: '',
                    po_status_dd: '',
                    concerns_on_order: '',
                    delivery_schedule_as_per_customer_po: '',
                    customer_clearance_for_billing: '',
                    reserved_quantity_from_stock: '',
                    mei_po_no: '',
                    month_of_delivery_scheduled: '',
                    category: '',
                    so_number: ''
                });
                const responseData = await response.json();
                console.log('Data submitted successfully');
            } else {
                const errorData = await response.json();
                console.error('Error submitting form:', await response.json());
            }
        } catch (error) {
            console.error('Error submitting form:', error);
        }
    };
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />
    <main class="flex-grow container mx-auto px-6 py-24 grid lg:grid-cols-1 gap-10">
        <form on:submit={handleSubmit} class="bg-white shadow-xl rounded-lg p-8 space-y-8">
            <h2 class="text-2xl font-semibold text-gray-900 mb-8">Enter Customer PO Data</h2>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
               
                <div>
                    <label for="sra_engineer_name" class="block text-sm font-medium text-gray-700">SRA Engineer Name</label>
                    <input type="text" 
                        id="sra_engineer_name"
                        bind:value={$formData.sra_engineer_name}
                        placeholder="Enter SRA Engineer Name"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                    <input type="text" 
                        id="supplier"
                        bind:value={$formData.supplier}
                        placeholder="Enter Supplier" 
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="customer" class="block text-sm font-medium text-gray-700">Customer Name</label>
                    <select
                        id="customer"
                        bind:value={$formData.customer_name}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a Customer</option>
                        {#each $customers as customer}
                            <option value={customer}>{customer}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="bs_no" class="block text-sm font-medium text-gray-700">BS Number</label>
                    <select
                        id="bs_no"
                        bind:value={$formData.bs_no}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a BS No</option>
                        {#each $bsNumbers as bsNumber}
                            <option value={bsNumber}>{bsNumber}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="customer_po_no" class="block text-sm font-medium text-gray-700">Customer P.O. No</label>
                    <input type="text" 
                        id="customer_po_no"
                        bind:value={$formData.customer_po_no}
                        placeholder="Enter Customer PO Number" 
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="po_date" class="block text-sm font-medium text-gray-700">PO Date</label>
                    <input type="date" 
                        id="po_date"
                        bind:value={$formData.po_date}
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
                        bind:value={$formData.part_code}
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
                        bind:value={$formData.quantity}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                    <select
                        id="unit"
                        bind:value={$formData.unit}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select a Unit</option>
                        {#each $units as unit}
                            <option value={unit}>{unit}</option>
                        {/each}
                    </select>
                </div>
                <div>
                    <label for="total_value" class="block text-sm font-medium text-gray-700">Total Value</label>
                    <input type="number"
                        id="total_value"
                        bind:value={$formData.total_value}
                        placeholder="Enter Total Value"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
                <div>
                    <label for="po_status_dd" class="block text-sm font-medium text-gray-700">PO Status</label>
                    <select
                        id="po_status_dd"
                        bind:value={$formData.po_status_dd}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    >
                        <option value="" disabled>Select PO Status</option>
                        {#each $poStatuses as status}
                            <option value={status}>{status}</option>
                        {/each}
                    </select>
                </div>
                <div>
                    <label for="concerns_on_order" class="block text-sm font-medium text-gray-700">Concerns on Order</label>
                    <select
                        id="concerns_on_order"
                        bind:value={$formData.concerns_on_order}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    >
                        <option value="" disabled>Select a Concern</option>
                        {#each $concernsOnOrders as concern}
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
                        bind:value={$formData.delivery_schedule_as_per_customer_po}
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
                        bind:value={$formData.customer_clearance_for_billing}
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
                        bind:value={$formData.reserved_quantity_from_stock}
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
                        bind:value={$formData.so_number}
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
                            bind:value={$formData.mei_po_no}
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
                    bind:value={$formData.month_of_delivery_scheduled}
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
                    bind:value={$formData.category}
                    placeholder="Enter Category"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required
                />
            </div>

            </div>
           

            <div class="flex justify-center mt-6">
                <button
                    type="submit"
                    class="px-8 py-3 bg-black text-white text-xl font-semibold rounded-lg hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-black transition duration-300"
                >
                    Submit
                </button>
        </form>
    </main>
</div>
