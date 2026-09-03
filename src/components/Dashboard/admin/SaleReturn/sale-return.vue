<template>
    <div class="min-h-screen bg-white dark:bg-slate-950 transition-colors duration-200">
        <AdminHeader
            :is-dark="isDark"
            @toggle-dark="toggleDarkMode"
            @toggle-menu="toggleMenu" />

        <div class="flex">
            <AdminNavbar
                :mobile-menu="mobileMenu"
                @close="mobileMenu = false" />

            
            <Message
                :successMsg="successMsg"
                :errorMsg="errorMsg"
                @update:successMsg="successMsg = $event"
                @update:errorMsg="errorMsg = $event"
            />
            
            <div class="flex-1 min-w-0 flex flex-col">
                <div class="min-h-screen bg-gray-50 dark:bg-[#0f172e]">
                    <div class="mx-auto px-4 sm:px-6 lg:px-8 py-5">

                        <!-- ================= MAIN GRID ================= -->
                        <div class="grid grid-cols-1 lg:grid-cols-12 gap-5 items-start">

                            <!-- ================= CART TABLE ================= -->
                            <div class="lg:col-span-8 flex flex-col gap-3">

                                <!-- Header -->
                                <div class="flex items-center justify-between">
                                    <label
                                        class="text-xs font-black uppercase tracking-wider text-slate-500 dark:text-slate-400 flex items-center gap-2">
                                        <i class="fa-solid fa-cart-shopping text-emerald-600 dark:text-orange-500"></i>
                                        Selected Order Items
                                    </label>

                                    <span
                                        class="text-[10px] font-bold px-2 py-0.5 rounded-full bg-emerald-50 text-emerald-700 border border-emerald-200 dark:bg-orange-500/10 dark:text-orange-400 dark:border-orange-500/20">
                                        {{ cartItems?.length || 0 }} Items Selected
                                    </span>
                                </div>

                                <!-- Table -->
                                <div
                                    class="w-full overflow-hidden rounded-2xl border border-slate-200/80 bg-white shadow-sm dark:border-slate-800 dark:bg-slate-900/40">

                                    <div class="max-h-screen overflow-y-auto overflow-x-auto custom-scrollbar">

                                        <table class="w-full text-left border-collapse">

                                            <thead
                                                class="sticky top-0 z-10 bg-slate-50 border-b border-slate-200 text-[10px] uppercase font-black text-slate-400 tracking-wider dark:bg-slate-900 dark:border-slate-800">

                                                <tr>
                                                    <th class="py-3 px-4">Product</th>
                                                    <th class="py-3 px-3 text-center">Qty</th>
                                                    <th class="py-3 px-3 text-center">Return Qty</th>
                                                    <th class="py-3 px-3 text-right">Unit Price</th>
                                                    <th class="py-3 px-3 text-right">Discount</th>
                                                    <th class="py-3 px-3 text-right">Subtotal</th>
                                                </tr>
                                            </thead>

                                            <tbody
                                                class="divide-y divide-slate-100 text-xs dark:divide-slate-800/60">

                                                <tr
                                                    v-for="item in cartItems"
                                                    :key="item.id"
                                                    class="group hover:bg-slate-50/80 transition-colors duration-200 dark:hover:bg-slate-800/40">

                                                    <!-- Product -->
                                                    <td class="py-3 px-4 min-w-[180px]">
                                                        <div class="flex items-center gap-3">

                                                            <div
                                                                @click="ProductDetails(item)"
                                                                class="relative w-10 h-10 rounded-lg overflow-hidden bg-slate-100 cursor-pointer flex-shrink-0 border border-slate-200 dark:bg-slate-800 dark:border-slate-700">

                                                                <img
                                                                    :src="getProductImage(item)"
                                                                    :alt="item.product?.name || 'Product Image'"
                                                                    class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
                                                                    @error="(e) => e.target.src = defaultProductImage"
                                                                />
                                                            </div>

                                                            <div class="flex flex-col min-w-0">

                                                                <h3
                                                                    @click="ProductDetails(item)"
                                                                    class="text-xs font-bold text-slate-800 cursor-pointer line-clamp-1 transition-colors group-hover:text-emerald-600 dark:text-slate-200 dark:group-hover:text-orange-400">

                                                                    {{ item.product?.name || 'N/A' }}
                                                                </h3>

                                                                <span
                                                                    v-if="item.product?.sku"
                                                                    class="text-[9px] text-slate-400">

                                                                    SKU: {{ item.product.sku }}
                                                                </span>

                                                                <span
                                                                    v-if="Number(item.discount) > 0"
                                                                    class="mt-0.5 text-[9px] font-extrabold text-green-600 dark:text-orange-400">

                                                                    Saved ৳{{
                                                                        formatMoney(
                                                                            Number(item.discount || 0) *
                                                                            Number(item.quantity || 0)
                                                                        )
                                                                    }}

                                                                    <template v-if="Number(item.point) > 0">
                                                                        - {{
                                                                            formatPoints(
                                                                                Number(item.point || 0) *
                                                                                Number(item.quantity || 0)
                                                                            )
                                                                        }} pts
                                                                    </template>
                                                                </span>

                                                            </div>
                                                        </div>
                                                    </td>

                                                    <!-- Quantity -->
                                                    <td class="py-3 px-3 text-center whitespace-nowrap">

                                                        <div class="inline-flex items-center p-0.5 rounded-lg bg-slate-100 border border-slate-200 dark:bg-slate-800 dark:border-slate-700">
                                                            <span
                                                                class="w-7 text-center text-xs font-bold text-slate-800 dark:text-slate-200">
                                                                {{ item.quantity }}
                                                            </span>
                                                        </div>
                                                    </td>

                                                    <!-- Return Quantity -->
                                                    <td class="py-3 px-3 text-center whitespace-nowrap">
                                                        <div class="inline-flex items-center p-0.5 rounded-lg bg-slate-100 border border-slate-200 dark:bg-slate-800 dark:border-slate-700">

                                                            <button
                                                                type="button"
                                                                @click="decreaseQty(item)"
                                                                :disabled="(item.returned_quantity || 0) <= 0"
                                                                class="w-5 h-5 flex items-center justify-center rounded bg-white text-slate-600 hover:text-emerald-600 disabled:opacity-40 dark:bg-slate-700 dark:text-slate-300 dark:hover:text-orange-400">
                                                                <i class="fa-solid fa-minus text-[8px]"></i>
                                                            </button>

                                                            <span class="w-7 text-center text-xs font-bold text-slate-800 dark:text-slate-200">
                                                                {{ item.returned_quantity || 0 }}
                                                            </span>

                                                            <button
                                                                type="button"
                                                                @click="increaseQty(item)"
                                                                :disabled="(item.returned_quantity || 0) >= item.quantity"
                                                                class="w-5 h-5 flex items-center justify-center rounded bg-white text-slate-600 hover:text-emerald-600 disabled:opacity-40 dark:bg-slate-700 dark:text-slate-300 dark:hover:text-orange-400">
                                                                <i class="fa-solid fa-plus text-[8px]"></i>
                                                            </button>
                                                        </div>
                                                    </td>

                                                    <!-- Price -->
                                                    <td class="py-3 px-3 text-right whitespace-nowrap">
                                                        <span class="text-xs font-semibold text-slate-700 dark:text-slate-300">
                                                            ৳{{ formatMoney(item.price) }}
                                                        </span>
                                                    </td>

                                                    <!-- Discount -->
                                                    <td class="py-3 px-3 text-right whitespace-nowrap">
                                                        <span class="text-xs font-semibold text-slate-700 dark:text-slate-300">
                                                            ৳{{ formatMoney(item.discount) }}
                                                        </span>
                                                    </td>

                                                    <!-- Total -->
                                                    <td
                                                        class="py-3 px-3 text-right whitespace-nowrap text-xs font-black text-slate-900 dark:text-white">
                                                        ৳{{ formatMoney(getItemTotal(item)) }}
                                                    </td>
                                                </tr>

                                                <!-- Empty -->
                                                <tr v-if="!cartItems?.length">
                                                    <td colspan="6" class="py-12 text-center">

                                                        <div
                                                            class="max-w-xs mx-auto flex flex-col items-center justify-center">

                                                            <div
                                                                class="mb-3 flex items-center justify-center w-14 h-14 rounded-2xl bg-slate-100 border border-slate-200 dark:bg-slate-800 dark:border-slate-700">

                                                                <i class="fa-solid fa-cart-plus text-2xl text-slate-400"></i>
                                                            </div>

                                                            <h3
                                                                class="text-sm font-semibold text-slate-800 dark:text-slate-200 mb-1">
                                                                Your cart is empty
                                                            </h3>

                                                            <p class="text-xs text-slate-500 dark:text-slate-400">
                                                                Looks like you haven't added any products.
                                                            </p>
                                                        </div>
                                                    </td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>


                                <!-- ================= CUSTOMER + PAYMENT ================= -->
                                <div class="bg-white dark:bg-[#0F172E] p-5 sm:p-6 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-5 mt-4">

                                    <div
                                        class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4">

                                        <h3
                                            class="text-base font-bold text-slate-900 dark:text-white flex items-center gap-2">

                                            <i
                                                class="fa-solid fa-receipt text-[#16a34a] dark:text-[#f97316]">
                                            </i>

                                            Payment Details & Customer Information
                                        </h3>
                                    </div>


                                    <!-- Customer -->
                                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">

                                        <div class="space-y-1.5">

                                            <label
                                                class="block text-xs font-semibold text-slate-600 dark:text-slate-300">
                                                Customer Phone
                                            </label>

                                            <input
                                                type="tel"
                                                v-model="form.phone_number"
                                                class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 bg-slate-50 text-sm text-slate-800 outline-none focus:ring-2 focus:ring-[#16a34a]/30 dark:border-slate-700 dark:bg-slate-800/50 dark:text-slate-200 dark:focus:ring-[#f97316]/30"
                                            />
                                        </div>

                                        <div class="space-y-1.5">

                                            <label
                                                class="block text-xs font-semibold text-slate-600 dark:text-slate-300">
                                                Customer Name
                                            </label>

                                            <input
                                                type="text"
                                                v-model="form.customer_name"
                                                class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 bg-slate-50 text-sm text-slate-800 outline-none focus:ring-2 focus:ring-[#16a34a]/30 dark:border-slate-700 dark:bg-slate-800/50 dark:text-slate-200 dark:focus:ring-[#f97316]/30"
                                            />
                                        </div>
                                    </div>


                                    <!-- Payment History -->
                                    <div
                                        v-if="payments?.length"
                                        class="border-t border-slate-100 dark:border-slate-800 pt-4 space-y-3">

                                        <h4
                                            class="text-xs font-bold text-slate-500 uppercase tracking-wider">
                                            Transaction Records
                                        </h4>

                                        <div
                                            class="divide-y divide-slate-100 dark:divide-slate-800/60 rounded-xl border border-slate-200/80 dark:border-slate-800 overflow-hidden">

                                            <div
                                                v-for="payment in payments"
                                                :key="payment.id"
                                                class="p-3 flex flex-wrap items-center justify-between gap-3 text-xs hover:bg-slate-50 dark:hover:bg-slate-800/30 transition-colors">

                                                <div class="space-y-1">

                                                    <div class="flex flex-wrap items-center gap-2">

                                                        <span
                                                            class="font-bold text-slate-900 dark:text-white text-sm">
                                                            ৳ {{ formatMoney(payment.amount) }}
                                                        </span>

                                                        <span
                                                            class="px-2 py-0.5 rounded-md text-[10px] font-semibold uppercase bg-emerald-100 text-emerald-800 dark:bg-emerald-500/20 dark:text-emerald-400">
                                                            {{ payment.payment_method || '-' }}
                                                        </span>

                                                        <span
                                                            class="px-2 py-0.5 rounded-md text-[10px] font-semibold uppercase bg-slate-100 text-slate-600 dark:bg-slate-800 dark:text-slate-300">
                                                            {{ payment.payment_type || '-' }}
                                                        </span>
                                                    </div>

                                                    <p class="text-slate-400 text-[11px]">
                                                        {{ formatDate(payment.paid_at) }}
                                                    </p>
                                                </div>


                                                <div
                                                    class="text-right space-y-1 text-[10px] text-slate-500 dark:text-slate-400">

                                                    <p v-if="payment.customer?.name">
                                                        Customer:
                                                        <strong class="text-slate-700 dark:text-slate-300">
                                                            {{ payment.customer.name }}
                                                        </strong>
                                                    </p>

                                                    <p v-if="payment.user?.name">
                                                        User:
                                                        <strong class="text-slate-700 dark:text-slate-300">
                                                            {{ payment.user.name }}
                                                        </strong>
                                                    </p>

                                                    <p v-if="payment.receiver?.name">
                                                        Receiver:
                                                        <strong class="text-slate-700 dark:text-slate-300">
                                                            {{ payment.receiver.name }}
                                                        </strong>
                                                    </p>

                                                    <p v-if="payment.verifier?.name">
                                                        Verifier:
                                                        <strong class="text-slate-700 dark:text-slate-300">
                                                            {{ payment.verifier.name }}
                                                        </strong>
                                                    </p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>


                            <!-- ================= ORDER INFORMATION ================= -->
                            <div class="lg:col-span-4">

                                <div
                                    class="bg-white dark:bg-[#0F172E] rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm p-5 space-y-3">

                                    <div
                                        class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-3">

                                        <h3
                                            class="text-sm font-bold text-slate-900 dark:text-white flex items-center gap-2">

                                            <i
                                                class="fa-solid fa-file-invoice text-[#16a34a] dark:text-[#f97316]">
                                            </i>

                                            Order Information
                                        </h3>

                                        <span
                                            class="text-[10px] font-bold px-2 py-1 rounded-md bg-slate-100 text-slate-500 dark:bg-slate-800 dark:text-slate-400">
                                            {{ order?.status || '-' }}
                                        </span>
                                    </div>


                                    <div class="space-y-2 text-xs">

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Order Reg</span>

                                            <span
                                                class="font-bold text-slate-800 dark:text-slate-200">
                                                {{ order?.reg || '-' }}
                                            </span>
                                        </div>

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Order Slug</span>

                                            <span
                                                class="font-semibold text-slate-700 dark:text-slate-300 truncate max-w-[60%]">
                                                {{ order?.slug || '-' }}
                                            </span>
                                        </div>

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Created At</span>

                                            <span
                                                class="font-semibold text-slate-700 dark:text-slate-300">
                                                {{ formatDate(order?.created_at) }}
                                            </span>
                                        </div>

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Created By</span>

                                            <span
                                                class="font-semibold text-slate-700 dark:text-slate-300">
                                                {{ order?.user?.name || '-' }}
                                            </span>
                                        </div>

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Customer</span>

                                            <span
                                                class="font-semibold text-slate-700 dark:text-slate-300">
                                                {{ order?.customer?.customer_name || '-' }}
                                            </span>
                                        </div>

                                        <div class="flex justify-between gap-3">
                                            <span class="text-slate-400">Phone</span>

                                            <span
                                                class="font-semibold text-slate-700 dark:text-slate-300">
                                                {{ order?.customer?.phone || '-' }}
                                            </span>
                                        </div>

                                    </div>
                                </div>


                                <!-- ================= ORDER SUMMARY ================= -->
                                <div class="lg:col-span-5 xl:col-span-4 mt-4">

                                    <div
                                        class="sticky top-6 bg-white dark:bg-[#0f172e] rounded-2xl p-5 sm:p-6 shadow-sm border border-slate-200/80 dark:border-slate-700/60 space-y-4">

                                        <!-- Header -->
                                        <div
                                            class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-3">

                                            <h2
                                                class="text-base font-black text-slate-900 dark:text-white flex items-center gap-2">

                                                Order Summary

                                                <span
                                                    class="w-2 h-2 rounded-full bg-[#16a34a] dark:bg-[#F97316] animate-pulse">
                                                </span>
                                            </h2>

                                            <span
                                                class="text-[#16a34a] dark:text-[#fb923c] font-bold bg-[#16a34a]/10 dark:bg-[#f97316]/20 px-2.5 py-1 rounded-lg text-[11px]">

                                                {{ formatPoints(totalPoint) }} pts
                                            </span>
                                        </div>


                                        <!-- CART SUMMARY -->
                                        <div class="space-y-2.5">

                                            <div class="flex justify-between text-sm">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Items
                                                </span>

                                                <span class="font-bold text-slate-800 dark:text-white">
                                                    {{ summary?.cart?.total_items || 0 }}
                                                </span>
                                            </div>

                                            <div class="flex justify-between text-sm">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Total Quantity
                                                </span>

                                                <span class="font-bold text-slate-800 dark:text-white">
                                                    {{ summary?.cart?.total_quantity || 0 }}
                                                </span>
                                            </div>

                                            <div class="flex justify-between text-sm">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Subtotal
                                                </span>

                                                <span class="font-bold text-slate-900 dark:text-white">
                                                    ৳{{ formatMoney(summary?.cart?.subtotal) }}
                                                </span>
                                            </div>

                                            <div class="flex justify-between text-sm">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Discount
                                                </span>

                                                <span class="text-red-500 font-bold">
                                                    - ৳{{ formatMoney(summary?.cart?.discount) }}
                                                </span>
                                            </div>

                                            <div class="h-px bg-slate-100 dark:bg-slate-700"></div>

                                            <div class="flex justify-between items-center">
                                                <span
                                                    class="text-sm font-bold text-slate-900 dark:text-white">
                                                    Total
                                                </span>

                                                <span
                                                    class="text-xl font-black text-[#16a34a] dark:text-[#F97316]">
                                                    ৳{{ formatMoney(summary?.cart?.total) }}
                                                </span>
                                            </div>
                                        </div>


                                        <div class="space-y-2.5">
                                            <div class="h-px bg-slate-100 dark:bg-slate-700"></div>
                                            
                                            <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-3">
                                                <h3 class="text-sm font-bold text-slate-900 dark:text-white flex items-center gap-2">
                                                    <i class="fa-solid fa-coins text-[#16a34a] dark:text-[#f97316]"></i>
                                                    Financial Breakdown
                                                </h3>

                                                <span
                                                    class="text-[10px] font-bold px-2 py-1 rounded-md"
                                                    :class="order?.status === 'partially_paid'
                                                        ? 'bg-amber-100 text-amber-700 dark:bg-amber-500/10 dark:text-amber-400'
                                                        : order?.status === 'paid'
                                                            ? 'bg-emerald-100 text-emerald-700 dark:bg-emerald-500/10 dark:text-emerald-400'
                                                            : 'bg-slate-100 text-slate-600 dark:bg-slate-800 dark:text-slate-300'">
                                                    {{ formatStatus(order?.status) }}
                                                </span>
                                            </div>

                                            <!-- Order Number / Currency -->
                                            <div class="grid grid-cols-2 gap-3 text-xs">
                                                <div>
                                                    <p class="text-slate-400 mb-0.5">Order Number</p>
                                                    <p class="font-bold text-slate-800 dark:text-slate-200 truncate">
                                                        {{ order?.order_number || '-' }}
                                                    </p>
                                                </div>
                                                <div>
                                                    <p class="text-slate-400 mb-0.5">Currency</p>
                                                    <p class="font-bold text-slate-800 dark:text-slate-200">
                                                        {{ order?.currency || '-' }}
                                                    </p>
                                                </div>
                                            </div>

                                            <div class="h-px bg-slate-100 dark:bg-slate-800"></div>

                                            <!-- Money Breakdown -->
                                            <div class="space-y-2 text-xs">

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">Subtotal</span>
                                                    <span class="font-bold text-slate-800 dark:text-slate-200">
                                                        ৳{{ formatMoney(order?.subtotal) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">Discount</span>
                                                    <span class="font-bold text-red-500">
                                                        - ৳{{ formatMoney(order?.discount) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">
                                                        VAT ({{ order?.vat_percentage || 0 }}%)
                                                    </span>
                                                    <span class="font-bold text-slate-800 dark:text-slate-200">
                                                        + ৳{{ formatMoney(order?.vat) }}
                                                    </span>
                                                </div>

                                                <div class="h-px bg-slate-100 dark:bg-slate-800"></div>

                                                <div class="flex justify-between items-center pt-1">
                                                    <span class="text-sm font-bold text-slate-900 dark:text-white">Payable Amount</span>
                                                    <span class="text-lg font-black text-[#16a34a] dark:text-[#F97316]">
                                                        ৳{{ formatMoney(order?.payable_amount) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">Due Amount</span>
                                                    <span
                                                        class="font-bold"
                                                        :class="Number(order?.due_amount) > 0 ? 'text-red-500' : 'text-emerald-600 dark:text-emerald-400'">
                                                        ৳{{ formatMoney(order?.due_amount) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">Refunded Amount</span>
                                                    <span class="font-bold text-red-500">
                                                        ৳{{ formatMoney(order?.refunded_amount) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-500 dark:text-slate-400">Earned Points</span>
                                                    <span class="font-bold text-slate-800 dark:text-slate-200">
                                                        {{ formatPoints(order?.point) }} pts
                                                    </span>
                                                </div>
                                            </div>

                                            <div class="h-px bg-slate-100 dark:bg-slate-800"></div>

                                            <!-- Payment Method & Timeline -->
                                            <div class="space-y-2 text-xs">

                                                <div class="flex justify-between">
                                                    <span class="text-slate-400">Payment Method</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300 capitalize">
                                                        {{ order?.payment_method || '-' }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-400">Order Date</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300">
                                                        {{ formatDate(order?.order_date) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between">
                                                    <span class="text-slate-400">Paid At</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300">
                                                        {{ order?.paid_at ? formatDate(order.paid_at) : '-' }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between" v-if="order?.completed_at">
                                                    <span class="text-slate-400">Completed At</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300">
                                                        {{ formatDate(order.completed_at) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between" v-if="order?.returned_at">
                                                    <span class="text-slate-400">Returned At</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300">
                                                        {{ formatDate(order.returned_at) }}
                                                    </span>
                                                </div>

                                                <div class="flex justify-between" v-if="order?.returned_by">
                                                    <span class="text-slate-400">Returned By</span>
                                                    <span class="font-semibold text-slate-700 dark:text-slate-300">
                                                        {{ order?.returned_by_user?.name || order?.returned_by }}
                                                    </span>
                                                </div>
                                            </div>

                                            <!-- Remarks -->
                                            <div v-if="order?.remarks" class="pt-2 border-t border-slate-100 dark:border-slate-800">
                                                <p class="text-slate-400 text-[10px] uppercase font-bold tracking-wider mb-1">Remarks</p>
                                                <p class="text-xs text-slate-600 dark:text-slate-300 bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2.5 leading-relaxed">
                                                    {{ order.remarks }}
                                                </p>
                                            </div>

                                            <!-- Technical Meta (collapsible) -->
                                            <details class="pt-2 border-t border-slate-100 dark:border-slate-800 group">
                                                <summary class="text-[10px] uppercase font-bold tracking-wider text-slate-400 cursor-pointer select-none flex items-center justify-between">
                                                    Technical Info
                                                    <i class="fa-solid fa-chevron-down text-[8px] transition-transform group-open:rotate-180"></i>
                                                </summary>

                                                <div class="mt-2 space-y-1.5 text-[11px]">
                                                    <div class="flex justify-between gap-3">
                                                        <span class="text-slate-400">IP Address</span>
                                                        <span class="font-mono text-slate-600 dark:text-slate-300">
                                                            {{ order?.ip_address || '-' }}
                                                        </span>
                                                    </div>
                                                    <div class="flex justify-between gap-3">
                                                        <span class="text-slate-400 flex-shrink-0">User Agent</span>
                                                        <span class="font-mono text-slate-600 dark:text-slate-300 truncate max-w-[65%] text-right" :title="order?.user_agent">
                                                            {{ order?.user_agent || '-' }}
                                                        </span>
                                                    </div>
                                                </div>
                                            </details>
                                        </div>

                                        <!-- PAYMENT SUMMARY -->
                                        <div
                                            class="border-t border-slate-100 dark:border-slate-800 pt-4 space-y-2">

                                            <p
                                                class="text-[10px] uppercase tracking-wider font-black text-slate-400">
                                                Payment Summary
                                            </p>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Total Paid
                                                </span>

                                                <strong class="text-slate-800 dark:text-white">
                                                    ৳{{ formatMoney(summary?.payment?.total_paid) }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Refunded
                                                </span>

                                                <strong class="text-red-500">
                                                    ৳{{ formatMoney(summary?.payment?.total_refunded) }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-sm pt-1">
                                                <span
                                                    class="font-bold text-slate-800 dark:text-white">
                                                    Net Received
                                                </span>

                                                <strong
                                                    class="text-[#16a34a] dark:text-[#F97316]">
                                                    ৳{{ formatMoney(summary?.payment?.net_received) }}
                                                </strong>
                                            </div>
                                        </div>


                                        <!-- RETURN SUMMARY -->
                                        <div
                                            class="border-t border-slate-100 dark:border-slate-800 pt-4 space-y-2">

                                            <p
                                                class="text-[10px] uppercase tracking-wider font-black text-slate-400">
                                                Return Summary
                                            </p>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Total Returns
                                                </span>

                                                <strong class="text-slate-800 dark:text-white">
                                                    {{ summary?.return?.total_returns || 0 }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Returned Quantity
                                                </span>

                                                <strong class="text-slate-800 dark:text-white">
                                                    {{ summary?.return?.returned_quantity || 0 }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Return Subtotal
                                                </span>

                                                <strong class="text-slate-800 dark:text-white">
                                                    ৳{{ formatMoney(summary?.return?.subtotal) }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Return Discount
                                                </span>

                                                <strong class="text-red-500">
                                                    ৳{{ formatMoney(summary?.return?.discount) }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-xs">
                                                <span class="text-slate-500 dark:text-slate-400">
                                                    Return VAT
                                                </span>

                                                <strong class="text-slate-800 dark:text-white">
                                                    ৳{{ formatMoney(summary?.return?.vat) }}
                                                </strong>
                                            </div>

                                            <div class="flex justify-between text-sm pt-1">
                                                <span class="font-bold text-red-500">
                                                    Refund Amount
                                                </span>

                                                <strong class="text-red-500">
                                                    ৳{{ formatMoney(summary?.return?.refund_amount) }}
                                                </strong>
                                            </div>
                                        </div>


                                        <!-- Checkout -->
                                        <button
                                            @click="handleCheckout"
                                            :disabled="checkoutLoading || !cartItems?.length"
                                            class="w-full mt-3 py-3 px-4 rounded-xl bg-[#16a34a] hover:bg-[#15803d] dark:bg-[#f97316] dark:hover:bg-[#ea580c] disabled:opacity-50 disabled:cursor-not-allowed text-white font-bold text-sm shadow-lg shadow-[#16a34a]/20 dark:shadow-[#f97316]/20 transition-all active:scale-[0.98] flex items-center justify-center gap-2">

                                            <span v-if="checkoutLoading">
                                                <i class="fa-solid fa-spinner fa-spin"></i>
                                                Processing...
                                            </span>

                                            <span v-else class="flex items-center gap-2">
                                                <i class="fa-solid fa-check text-xs"></i>
                                                Complete Checkout
                                            </span>
                                        </button>

                                    </div>
                                </div>
                                

                            </div>
                        </div>


                        <!-- ================= PAYMENT + RETURN + SUMMARY ================= -->
                        <div class="grid grid-cols-1 lg:grid-cols-12 gap-5 items-start mt-5">

                            <!-- LEFT SIDE -->
                            <div class="lg:col-span-7 xl:col-span-8 space-y-5">

                                <!-- ================= RETURN HISTORY ================= -->
                                <div
                                    v-if="returns?.length"
                                    class="bg-white dark:bg-[#0F172E] p-5 sm:p-6 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm">

                                    <div
                                        class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-4">

                                        <h3
                                            class="text-base font-bold text-slate-900 dark:text-white flex items-center gap-2">

                                            <i class="fa-solid fa-arrow-rotate-left text-red-500"></i>
                                            Return History
                                        </h3>

                                        <span class="text-xs text-slate-400">
                                            {{ returns.length }} Return(s)
                                        </span>
                                    </div>


                                    <div class="space-y-3">

                                        <div
                                            v-for="returnItem in returns"
                                            :key="returnItem.id"
                                            class="border border-slate-200 dark:border-slate-800 rounded-xl p-4">

                                            <!-- Return Summary -->
                                            <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-xs mb-3">

                                                <div>
                                                    <p class="text-slate-400">Subtotal</p>
                                                    <p class="font-bold text-slate-800 dark:text-white">
                                                        ৳{{ formatMoney(returnItem.subtotal) }}
                                                    </p>
                                                </div>

                                                <div>
                                                    <p class="text-slate-400">Discount</p>
                                                    <p class="font-bold text-red-500">
                                                        ৳{{ formatMoney(returnItem.discount) }}
                                                    </p>
                                                </div>

                                                <div>
                                                    <p class="text-slate-400">VAT</p>
                                                    <p class="font-bold text-slate-800 dark:text-white">
                                                        ৳{{ formatMoney(returnItem.vat) }}
                                                    </p>
                                                </div>

                                                <div>
                                                    <p class="text-slate-400">Refund</p>
                                                    <p class="font-black text-red-500">
                                                        ৳{{ formatMoney(returnItem.refund_amount) }}
                                                    </p>
                                                </div>
                                            </div>


                                            <!-- Return Items -->
                                            <div
                                                v-for="item in getReturnItems(returnItem)"
                                                :key="item.id"
                                                class="flex items-center justify-between gap-3 border-t border-slate-100 dark:border-slate-800 py-3 text-xs">

                                                <div class="flex items-center gap-3">

                                                    <img
                                                        :src="getReturnProductImage(item)"
                                                        class="w-9 h-9 rounded-lg object-cover bg-slate-100 dark:bg-slate-800"
                                                        @error="(e) => e.target.src = defaultProductImage"
                                                    />

                                                    <div>
                                                        <p
                                                            class="font-bold text-slate-800 dark:text-white">
                                                            {{ item.product?.name || 'N/A' }}
                                                        </p>

                                                        <p class="text-[10px] text-slate-400">
                                                            Batch: {{ item.stock?.batch_no || '-' }}
                                                        </p>
                                                    </div>
                                                </div>

                                                <span
                                                    class="font-bold text-slate-700 dark:text-slate-300">
                                                    Qty: {{ item.quantity }}
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>


                            
                        </div>
                    </div>

                </div>
            </div>

        </div>
    </div>
    <FooterSection />
</template>
<script setup>
import { computed, onMounted, reactive, ref } from "vue";
import api from "../../../../services/api.js";
import { useRouter, useRoute } from "vue-router";

import AdminNavbar from "../admin-navbar.vue";
import AdminHeader from "../admin-header.vue";
import Message from "../../../Message/message.vue";
import FooterSection from "../../../footer.vue";

const router = useRouter();
const route = useRoute();

const mobileMenu = ref(false);
const loading = ref(false);
const checkoutLoading = ref(false);
const saving = ref(false);

const errorMsg = ref("");
const successMsg = ref("");

// =====================================
// Mobile Menu
// =====================================

function toggleMenu() {
    mobileMenu.value = !mobileMenu.value;
}

























// =====================================
// Backend Main Data
// =====================================

const order = ref(null);
const cartItems = ref([]);
const payments = ref([]);
const returns = ref([]);

// =====================================
// Backend Summary
// =====================================

const summary = reactive({
    cart: {
        total_items: 0,
        total_quantity: 0,
        subtotal: 0,
        discount: 0,
        total: 0,
    },

    payment: {
        total_paid: 0,
        total_refunded: 0,
        net_received: 0,
    },

    return: {
        total_returns: 0,
        returned_quantity: 0,
        subtotal: 0,
        discount: 0,
        vat: 0,
        refund_amount: 0,
    },
});

// =====================================
// Form
// =====================================

const form = reactive({
    customer_name: "",
    phone_number: "",

    payment_method: "cash",

    vat: 0,
    discount: 0,

    received_amount: 0,
});

// =====================================
// Payment Methods
// =====================================

const paymentMethods = [
    { id: "cash", label: "Cash" },
    { id: "bkash", label: "bKash" },
    { id: "nagad", label: "Nagad" },
    { id: "rocket", label: "Rocket" },
    { id: "card", label: "Card" },
    { id: "bank", label: "Bank" },
    { id: "wallet", label: "Wallet" },
];

// =====================================
// Get All Backend Data
// =====================================

async function getCartItems() {
    loading.value = true;
    errorMsg.value = "";

    const reg = route.params.reg;
    const slug = route.params.slug;

    try {
        const res = await api.get(
            `admin/return/cart/${reg}/${slug}`
        );

        if (!res.data?.success) {
            throw new Error(
                res.data?.message ||
                "Unable to fetch order details."
            );
        }

        const data = res.data?.data || {};
        const apiSummary = res.data?.summary || {};

        // =================================
        // Order
        // =================================

        order.value = data.order || null;

        // =================================
        // Cart Items
        // =================================

        cartItems.value = Array.isArray(
            data.cart_items
        )
            ? data.cart_items
            : [];

        // =================================
        // Payments
        // =================================

        payments.value = Array.isArray(
            data.payments
        )
            ? data.payments
            : [];

        // =================================
        // Returns
        // =================================

        returns.value = Array.isArray(
            data.returns
        )
            ? data.returns
            : [];

        // =================================
        // Cart Summary
        // =================================

        Object.assign(
            summary.cart,
            apiSummary.cart || {}
        );

        // =================================
        // Payment Summary
        // =================================

        Object.assign(
            summary.payment,
            apiSummary.payment || {}
        );

        // =================================
        // Return Summary
        // =================================

        Object.assign(
            summary.return,
            apiSummary.return || {}
        );

        // =================================
        // Customer Data
        // =================================

        if (order.value?.customer) {
            form.customer_name = order.value.customer?.customer_name || "";
            form.phone_number = order.value.customer?.phone || order.value.customer?.phone_number || "";
        }

        // =================================
        // Order Payment Method
        // =================================

        if (order.value?.payment_method) {
            form.payment_method =
                order.value.payment_method;
        }

        // =================================
        // Optional Order VAT
        // =================================

        if (order.value?.vat !== undefined) {
            form.vat =
                Number(order.value.vat) || 0;
        }

        // =================================
        // Optional Order Discount
        // =================================

        if (order.value?.discount !== undefined) {
            form.discount =
                Number(order.value.discount) || 0;
        }

    } catch (err) {
        console.error(
            "Order Details Fetch Error:",
            err
        );

        errorMsg.value =
            err?.response?.data?.message ||
            err?.message ||
            "Something went wrong.";

        order.value = null;
        cartItems.value = [];
        payments.value = [];
        returns.value = [];

    } finally {
        loading.value = false;
    }
}

// =====================================
// Cart Registration
// =====================================

const cartReg = computed(() => {
    return order.value?.reg ||
        route.params.reg ||
        null;
});

// =====================================
// Order Basic Data Helpers
// =====================================

const orderReg = computed(() => {
    return order.value?.reg || "";
});

const orderSlug = computed(() => {
    return order.value?.slug || "";
});

const orderCustomer = computed(() => {
    return order.value?.customer || null;
});

const orderUser = computed(() => {
    return order.value?.user || null;
});

const formatStatus = (status) => {
    if (!status) return '-';
    return status
        .split('_')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ');
};

// =====================================
// Money Helpers
// =====================================

const roundMoney = (value) => {
    const number = Number(value);

    if (!Number.isFinite(number)) {
        return 0;
    }

    return Math.round(
        (number + Number.EPSILON) * 100
    ) / 100;
};

const formatMoney = (value) => {
    return roundMoney(value).toLocaleString(
        "en-BD",
        {
            minimumFractionDigits: 2,
            maximumFractionDigits: 2,
        }
    );
};

const formatPoints = (value) => {
    const number = Number(value) || 0;

    return number.toLocaleString(
        "en-BD",
        {
            maximumFractionDigits: 2,
        }
    );
};

const formatDate = (value) => {
    if (!value) return "-";

    const date = new Date(value);

    if (Number.isNaN(date.getTime())) {
        return value;
    }

    return date.toLocaleString("en-BD");
};

// =====================================
// Product Image
// =====================================

const defaultProductImage =
    "/images/product/default-product.webp";

const getProductImage = (item) => {
    const images = item?.product?.images;

    if (
        Array.isArray(images) &&
        images.length > 0
    ) {
        return images[0]?.url ||
            defaultProductImage;
    }

    return defaultProductImage;
};

// =====================================
// Cart Item Calculations
// =====================================

const getItemSubtotal = (item) => {
    const price =
        Number(item?.price) || 0;

    const quantity =
        Number(item?.quantity) || 0;

    return roundMoney(
        price * quantity
    );
};

const getItemDiscount = (item) => {
    const discount =
        Number(item?.discount) || 0;

    const quantity =
        Number(item?.quantity) || 0;

    return roundMoney(
        discount * quantity
    );
};

const getItemTotal = (item) => {
    return roundMoney(
        Math.max(
            0,
            getItemSubtotal(item) -
            getItemDiscount(item)
        )
    );
};

const getItemPoint = (item) => {
    const point =
        Number(item?.point) || 0;

    const quantity =
        Number(item?.quantity) || 0;

    return roundMoney(
        point * quantity
    );
};

// =====================================
// Frontend Calculations
// Backend summary also available
// =====================================

const subtotal = computed(() => {
    return roundMoney(
        cartItems.value.reduce(
            (sum, item) => {
                return sum +
                    getItemSubtotal(item);
            },
            0
        )
    );
});

const productDiscount = computed(() => {
    return roundMoney(
        cartItems.value.reduce(
            (sum, item) => {
                return sum +
                    getItemDiscount(item);
            },
            0
        )
    );
});

const productTotal = computed(() => {
    return roundMoney(
        Math.max(
            0,
            subtotal.value -
            productDiscount.value
        )
    );
});

const totalPoint = computed(() => {
    return roundMoney(
        cartItems.value.reduce(
            (sum, item) => {
                return sum +
                    getItemPoint(item);
            },
            0
        )
    );
});

const totalQuantity = computed(() => {
    return cartItems.value.reduce(
        (sum, item) => {
            return sum +
                (Number(item?.quantity) || 0);
        },
        0
    );
});

// =====================================
// Manual Discount
// =====================================

const manualDiscount = computed(() => {
    const discount =
        Number(form.discount) || 0;

    return roundMoney(
        Math.min(
            Math.max(0, discount),
            productTotal.value
        )
    );
});

// =====================================
// VAT
// =====================================

const vatAmount = computed(() => {
    const vatRate =
        Math.max(
            0,
            Number(form.vat) || 0
        );

    const taxableAmount =
        Math.max(
            0,
            productTotal.value -
            manualDiscount.value
        );

    return roundMoney(
        taxableAmount *
        vatRate /
        100
    );
});

// =====================================
// Payable
// =====================================

const totalPayable = computed(() => {
    return roundMoney(
        Math.max(
            0,
            productTotal.value -
            manualDiscount.value +
            vatAmount.value
        )
    );
});

// =====================================
// Received Amount
// =====================================

const receivedAmount = computed(() => {
    return roundMoney(
        Math.max(
            0,
            Number(form.received_amount) || 0
        )
    );
});

// =====================================
// Change Amount
// =====================================

const changeAmount = computed(() => {
    return roundMoney(
        Math.max(
            0,
            receivedAmount.value -
            totalPayable.value
        )
    );
});

// =====================================
// Due Amount
// =====================================

const dueAmount = computed(() => {
    return roundMoney(
        Math.max(
            0,
            totalPayable.value -
            receivedAmount.value
        )
    );
});

// =====================================
// Backend Payment Summary
// =====================================

const totalPaid = computed(() => {
    return roundMoney(
        summary.payment?.total_paid
    );
});

const totalRefunded = computed(() => {
    return roundMoney(
        summary.payment?.total_refunded
    );
});

const netReceived = computed(() => {
    return roundMoney(
        summary.payment?.net_received
    );
});

// =====================================
// Backend Return Summary
// =====================================

const totalReturns = computed(() => {
    return Number(
        summary.return?.total_returns
    ) || 0;
});

const returnedQuantity = computed(() => {
    return Number(
        summary.return?.returned_quantity
    ) || 0;
});

const returnSubtotal = computed(() => {
    return roundMoney(
        summary.return?.subtotal
    );
});

const returnDiscount = computed(() => {
    return roundMoney(
        summary.return?.discount
    );
});

const returnVat = computed(() => {
    return roundMoney(
        summary.return?.vat
    );
});

const refundAmount = computed(() => {
    return roundMoney(
        summary.return?.refund_amount
    );
});

// =====================================
// Payment Helpers
// =====================================

const getPaymentCustomer = (payment) => {
    return payment?.customer || null;
};

const getPaymentUser = (payment) => {
    return payment?.user || null;
};

const getPaymentReceiver = (payment) => {
    return payment?.receiver || null;
};

const getPaymentVerifier = (payment) => {
    return payment?.verifier || null;
};

// =====================================
// Return Helpers
// =====================================

const getReturnItems = (returnItem) => {
    return Array.isArray(
        returnItem?.items
    )
        ? returnItem.items
        : [];
};

const getReturnUser = (returnItem) => {
    return returnItem?.user || null;
};

const getReturnCustomer = (returnItem) => {
    return returnItem?.customer || null;
};

const getApprovedBy = (returnItem) => {
    return returnItem?.approved_by || null;
};

const getReturnProductImage = (item) => {
    const images =
        item?.product?.images;

    if (
        Array.isArray(images) &&
        images.length > 0
    ) {
        return images[0]?.url ||
            defaultProductImage;
    }

    return defaultProductImage;
};


















// qty update
const qtyTimers = {};

// 1. Return quantity increase (max = cart.quantity)
function increaseQty(item) {
    const current = Number(item.returned_quantity || 0);
    const max = Number(item.quantity);
    if (current < max) {
        item.returned_quantity = current + 1;
        queueQtyUpdate(item);
    }
}

// 2. Return quantity decrease (min = 0)
function decreaseQty(item) {
    const current = Number(item.returned_quantity || 0);
    if (current > 0) {
        item.returned_quantity = current - 1;
        queueQtyUpdate(item);
    }
}

// 3. Debounce
function queueQtyUpdate(item) {
    const key = `${item.reg}_${item.product_id}`;
    if (qtyTimers[key]) clearTimeout(qtyTimers[key]);
    qtyTimers[key] = setTimeout(() => {
        updateQty(item);
    }, 500);
}


async function updateQty(item) {
    try {
        const res = await api.post(`/admin/return/cart/qty-update/${item.reg}/${item.product_id}`, {
            quantity: Number(item.returned_quantity),
        });
        if (res?.data?.success) {
            item.returned_quantity = Number(res.data.data.returned_quantity);
            item.quantity = Number(res.data.data.quantity);
            if (res.data.data.available_return_quantity !== undefined) {
                item.available_return_quantity = res.data.data.available_return_quantity;
            }
        }
        await getCartItems();
    } catch (err) {
        await getCartItems();
        const msg = err?.response?.data?.message || "Something went wrong or Out of stock.";
        errorMsg.value = msg;
        setTimeout(() => {
            errorMsg.value = "";
        }, 3000);
    }
}





















// =====================================
// Dark Mode
// =====================================

const isDark = ref(
    localStorage.getItem("theme") === "dark"
);

function applyTheme(dark) {
    isDark.value = dark;

    document.documentElement.classList.toggle(
        "dark",
        dark
    );

    localStorage.setItem(
        "theme",
        dark ? "dark" : "light"
    );
}

function toggleDarkMode() {
    applyTheme(!isDark.value);
}

// =====================================
// Mounted
// =====================================

onMounted(() => {
    applyTheme(isDark.value);
    getCartItems();
});
</script>

<style scoped>
.input{
    @apply w-full rounded-xl border border-slate-200 bg-white px-3 py-2.5 text-sm text-slate-700 placeholder:text-slate-400
        focus:outline-none focus:ring-2 focus:ring-indigo-500
        dark:border-slate-700 dark:bg-slate-950 dark:text-slate-100;
}
.inputDisabled{
    @apply w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2.5 text-sm text-slate-600
        dark:border-slate-700 dark:bg-slate-800 dark:text-slate-300;
}
</style>