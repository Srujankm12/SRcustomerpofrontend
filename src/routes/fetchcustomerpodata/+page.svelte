<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let isLoading = true;
    let expandedRow = null;
    let showDownloadMessage = false;
    const fetchurl = "http://localhost:8000/fetch";
    const downloadexcelurl = "http://localhost:8000/download";


    const downloadexcelcpo = async () => {
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
    onMount(async () => {
        try {
            const response = await fetch(fetchurl);
            if (response.ok) {

                data = await response.json();
                console.log(data);
                isLoading = false;
            } else {
                console.error("Error fetching data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        }
    });

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
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />

    <main class="flex-grow py-28 px-2">
        {#if isLoading}
            <div class="flex justify-center items-center h-full">
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800 border-gray-300"></div>
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
                            {#each data as row, index}
                            <tr class="border-t border-gray-300 hover:bg-gray-200">
                                <td class="py-3 px-4 text-center">{row.id}</td>

                                <td class="py-3 px-4 text-center">{row.sra_engineer_name}</td>
                                <td class="py-3 px-4 text-center">{row.supplier}</td>
                                <td class="py-3 px-4 text-center">{row.customer_name}</td>
                                <td class="py-3 px-4 text-center">{row.bs_no}</td>
                                <td class="py-3 px-4 text-center">{row.customer_po_no}</td>
                                <td class="py-3 px-4 text-center">{row.po_date ? new Date(row.po_date).toLocaleDateString() : "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.part_code}</td>
                                <td class="py-3 px-4 text-center">{row.quantity}</td>
                                <td class="py-3 px-4 text-center">{row.unit}</td>
                                <td class="py-3 px-4 text-center">₹{row.total_value ? row.total_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">{row.po_status_dd}</td>
                                <td class="py-3 px-4 text-center">{row.concerns_on_order}</td>
                                <td class="py-3 px-4 text-center">₹{row.billable_scheduled_value ? row.billable_scheduled_value.toFixed(2) : "0.00"}</td>
                                <td class="py-3 px-4 text-center">{row.delivery_schedule_as_per_customer_po ?? "N/A"}</td>
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
                                <td class="py-3 px-4 text-center">{row.month_of_delivery_scheduled ?? "N/A"}</td>
                                <td class="py-3 px-4 text-center">{row.category}</td>
                        
                                <td class="py-3 px-4 text-center">
                                        <button
                                            class="bg-black text-white text-xs px-2 py-1 rounded hover:bg-gray-800 transition"
                                            on:click={() => expandedRow = expandedRow === index ? null : index}>
                                            {expandedRow === index ? "Hide" : "View"}
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
                on:click={downloadexcelcpo}
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
  
</div>
