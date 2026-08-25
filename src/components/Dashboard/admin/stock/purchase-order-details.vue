<template>
    <div class="min-h-screen bg-white dark:bg-slate-950 transition-colors duration-200">
        <HeaderSection
            :is-dark="isDark"
            @toggle-dark="toggleDarkMode"
            @toggle-menu="toggleMenu"
        />

        <div class="flex  min-h-[calc(100vh-56px)]">
            <Navbar
                :mobile-menu="mobileMenu"
                @close="mobileMenu = false" />

            <Message
                :successMsg="successMsg"
                :errorMsg="errorMsg"
                @update:successMsg="successMsg = $event"
                @update:errorMsg="errorMsg = $event"
            />

            <!-- Content -->
            <div class="flex-1 min-w-0">
                <main class="flex-1 min-h-screen min-w-0 bg-gray-50 dark:bg-[#0C1326] px-4 sm:px-6 lg:px-8 py-6 transition-colors duration-300">

                    <!-- Loading skeleton -->
                    <div v-if="loading && !order" class="space-y-6 animate-pulse">
                        <div class="flex items-center justify-between pb-5 border-b border-slate-200 dark:border-slate-800/60">
                            <div class="flex items-center gap-4">
                                <div class="w-10 h-10 rounded-xl bg-slate-200 dark:bg-slate-800"></div>
                                <div class="space-y-2">
                                    <div class="h-5 w-40 bg-slate-200 dark:bg-slate-800 rounded"></div>
                                    <div class="h-3 w-56 bg-slate-200 dark:bg-slate-800 rounded"></div>
                                </div>
                            </div>
                            <div class="flex gap-3">
                                <div class="h-9 w-32 bg-slate-200 dark:bg-slate-800 rounded-lg"></div>
                                <div class="h-9 w-32 bg-slate-200 dark:bg-slate-800 rounded-lg"></div>
                            </div>
                        </div>
                        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                            <div class="lg:col-span-2 space-y-6">
                                <div class="grid grid-cols-1 sm:grid-cols-4 gap-4">
                                    <div v-for="n in 4" :key="n" class="h-24 bg-slate-200 dark:bg-slate-800 rounded-xl"></div>
                                </div>
                                <div class="h-32 bg-slate-200 dark:bg-slate-800 rounded-2xl"></div>
                                <div class="h-72 bg-slate-200 dark:bg-slate-800 rounded-2xl"></div>
                                <div class="h-48 bg-slate-200 dark:bg-slate-800 rounded-2xl"></div>
                            </div>
                            <div class="h-96 bg-slate-200 dark:bg-slate-800 rounded-2xl"></div>
                        </div>
                    </div>

                    <!-- Not found -->
                    <div v-else-if="!order" class="flex flex-col items-center text-center py-24">
                        <div class="w-14 h-14 rounded-2xl bg-slate-50 dark:bg-slate-800/60 flex items-center justify-center mb-4">
                            <i class="fa-solid fa-circle-exclamation text-2xl text-slate-400 dark:text-slate-500"></i>
                        </div>
                        <p class="text-sm font-semibold text-slate-700 dark:text-slate-300">Order not found</p>
                        <p class="text-xs text-slate-400 dark:text-slate-500 mt-1 max-w-xs leading-relaxed">
                            {{ errorMsg || 'This order may not exist or may have been removed.' }}
                        </p>
                        <button @click="$router.back()" class="mt-5 px-4 py-2 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-lg text-sm font-semibold text-slate-700 dark:text-slate-300 hover:bg-slate-50 transition shadow-sm">
                            Go back
                        </button>
                    </div>

                    
                    <div v-else>

                        <!-- Page Header -->
                        <div
                            class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 mb-6 pb-5 border-b border-slate-200 dark:border-slate-800/60">

                            <div class="flex items-center gap-4">

                                <button
                                    @click="$router.back()"
                                    class="p-2 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl hover:bg-slate-50 dark:hover:bg-slate-800 transition shadow-sm shrink-0"
                                >
                                    <i class="fa-solid fa-arrow-left-long text-slate-600 dark:text-slate-400"></i>
                                </button>

                                <div class="space-y-1.5">

                                    <div class="flex flex-wrap items-center gap-2.5">

                                        <h1
                                            class="text-xl font-bold tracking-tight text-slate-900 dark:text-white flex items-center gap-1.5"
                                        >
                                            Order
                                            <span class="text-green-600 dark:text-orange-400">
                                                #{{ order.order_number }}
                                            </span>
                                        </h1>

                                        <span
                                            v-if="Number(order.discount) > 0"
                                            class="inline-flex items-center gap-1 bg-amber-50 dark:bg-amber-950/40 text-amber-700 dark:text-amber-400 font-mono text-xs font-semibold px-2 py-0.5 rounded-full border border-amber-200/60 dark:border-amber-900/50 uppercase tracking-wider"
                                        >
                                            <i class="fa-solid fa-tags text-amber-500"></i>
                                            Discount Applied
                                        </span>

                                    </div>

                                    <p
                                        class="text-sm font-medium text-slate-500 dark:text-slate-400 flex items-center gap-1.5"
                                    >
                                        <i class="fa-regular fa-calendar text-slate-400 dark:text-slate-500"></i>

                                        Placed on {{ formatDate(order.order_date) }}
                                    </p>

                                </div>
                            </div>

                            <div class="flex items-center gap-3">

                                <!-- <button
                                    type="button" @click="downloadInvoice" :disabled="invoiceDownloading"
                                    class="px-4 py-2 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-lg text-sm font-semibold text-slate-700 dark:text-slate-300 hover:bg-slate-50 dark:hover:bg-slate-800 disabled:opacity-50 transition shadow-sm"
                                >
                                    <i class="fa-solid fa-download mr-1.5"></i>
                                    Download Invoice
                                </button> -->

                                <button
                                    type="button"
                                    @click="printOrder(order)"
                                    class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-lg text-sm font-semibold shadow-md shadow-indigo-500/20 transition"
                                >
                                    <i class="fa-solid fa-print mr-1.5"></i>
                                    Print Details
                                </button>

                            </div>
                        </div>


                        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">

                            <!-- LEFT -->
                            <div class="lg:col-span-2 space-y-6">

                                <!-- Metric Cards -->
                                <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4">

                                    <!-- Payment Date -->
                                    <div class="bg-white dark:bg-slate-900 p-5 rounded-xl border border-slate-200 dark:border-slate-800 shadow-sm">
                                        <div class="flex items-center gap-2 mb-3">
                                            <i class="fa-regular fa-calendar text-slate-400 text-sm"></i>

                                            <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">
                                                Payment Date
                                            </p>
                                        </div>

                                        <p class="text-sm font-semibold text-slate-700 dark:text-slate-300">
                                            {{ order.paid_at ? formatDate(order.paid_at) : 'Not Paid Yet' }}
                                        </p>
                                    </div>


                                    <!-- Total Amount -->
                                    <div
                                        class="bg-white dark:bg-slate-900 p-5 rounded-xl border border-slate-200 dark:border-slate-800 shadow-sm"
                                    >
                                        <div class="flex items-center gap-2 mb-3">
                                            <i class="fa-solid fa-wallet text-slate-400 text-sm"></i>

                                            <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">
                                                Total Amount
                                            </p>
                                        </div>

                                        <p class="text-2xl font-bold font-mono text-slate-900 dark:text-white">
                                            {{ money(order.payable_amount) }}
                                        </p>
                                    </div>


                                    <!-- Order Status -->
                                    <div
                                        class="bg-white dark:bg-slate-900 p-5 rounded-xl border border-slate-200 dark:border-slate-800 shadow-sm"
                                    >
                                        <div class="flex justify-between items-center mb-3">

                                            <div class="flex items-center gap-2">
                                                <i class="fa-solid fa-truck-fast text-slate-400 text-sm"></i>

                                                <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">
                                                    Order Status
                                                </p>
                                            </div>

                                        </div>

                                        <span
                                            class="px-3 py-1 rounded-lg text-[11px] font-bold uppercase inline-flex items-center gap-2 border w-fit"
                                            :class="statusClass(order.status)"
                                        >
                                            <span class="h-2 w-2 rounded-full bg-current"></span>

                                            {{ formatStatus(order.status) }}
                                        </span>
                                    </div>


                                    <!-- Due Amount -->
                                    <div
                                        class="bg-white dark:bg-slate-900
                                            p-5 rounded-xl
                                            border border-slate-200 dark:border-slate-800
                                            shadow-sm"
                                    >
                                        <div class="flex items-center gap-2 mb-3">
                                            <i class="fa-solid fa-hourglass-half text-slate-400 text-sm"></i>

                                            <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">
                                                Due Amount
                                            </p>
                                        </div>

                                        <p
                                            class="text-2xl font-bold font-mono"
                                            :class="Number(order.due_amount || 0) > 0
                                                ? 'text-amber-600 dark:text-amber-400'
                                                : 'text-emerald-600 dark:text-emerald-400'"
                                        >
                                            {{ money(order.due_amount) }}
                                        </p>
                                    </div>

                                </div>


                                <!-- Transaction Details -->
                                <div
                                    class="bg-white dark:bg-slate-900 rounded-2xl border border-slate-200 dark:border-slate-800 shadow-sm overflow-hidden">

                                    <div
                                        class="px-6 py-5 border-b border-slate-100 dark:border-slate-800 flex items-center justify-between"
                                    >
                                        <div>
                                            <h3 class="font-bold text-base text-slate-900 dark:text-white">
                                                Transaction Details
                                            </h3>

                                            <p class="text-xs text-slate-400 dark:text-slate-500 mt-0.5">
                                                Order receipt
                                            </p>
                                        </div>

                                        <span
                                            class="inline-flex items-center gap-1.5 text-xs font-semibold px-3 py-1.5 rounded-full bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 uppercase"
                                        >
                                            {{ order.payment_method }}
                                        </span>
                                    </div>


                                    <div class="px-6 py-4 space-y-3">

                                        <div class="flex justify-between items-center">
                                            <span class="text-sm text-slate-500 dark:text-slate-400">
                                                Registration Number
                                            </span>

                                            <span class="text-sm font-semibold text-slate-900 dark:text-white">
                                                {{ order.reg }}
                                            </span>
                                        </div>


                                        <div class="flex justify-between items-center">
                                            <span class="text-sm text-slate-500 dark:text-slate-400">
                                                Order Number
                                            </span>

                                            <span class="text-sm font-semibold text-slate-900 dark:text-white">
                                                {{ order.order_number }}
                                            </span>
                                        </div>


                                        <div class="flex justify-between items-center">
                                            <span class="text-sm text-slate-500 dark:text-slate-400">
                                                Currency
                                            </span>

                                            <span class="text-sm font-semibold text-slate-900 dark:text-white">
                                                {{ order.currency }} — Bangladeshi Taka
                                            </span>
                                        </div>


                                        <div class="flex justify-between items-center">
                                            <span class="text-sm text-slate-500 dark:text-slate-400">
                                                Payment Method
                                            </span>

                                            <span class="text-sm font-semibold text-slate-900 dark:text-white capitalize">
                                                {{ order.payment_method }}
                                            </span>
                                        </div>

                                    </div>


                                    <div class="relative px-6">
                                        <div class="border-t border-dashed border-slate-200 dark:border-slate-700"></div>
                                    </div>


                                    <!-- Amount Breakdown -->
                                    <div class="px-6 pt-5 pb-6">

                                        <h4
                                            class="text-xs font-bold uppercase tracking-wide
                                                text-slate-400 dark:text-slate-500 mb-3"
                                        >
                                            Amount Breakdown
                                        </h4>

                                        <div class="space-y-2 text-sm">

                                            <!-- Subtotal -->
                                            <div class="flex justify-between text-slate-500 dark:text-slate-400">
                                                <span>Subtotal</span>

                                                <span class="font-medium text-slate-700 dark:text-slate-300">
                                                    {{ money(order.subtotal) }}
                                                </span>
                                            </div>


                                            <!-- Discount -->
                                            <div
                                                v-if="Number(order.discount || 0) > 0"
                                                class="flex justify-between text-slate-500 dark:text-slate-400"
                                            >
                                                <span>Discount</span>

                                                <span class="font-medium text-emerald-600 dark:text-emerald-400">
                                                    − {{ money(order.discount) }}
                                                </span>
                                            </div>


                                            <!-- VAT -->
                                            <div
                                                v-if="Number(order.vat || 0) > 0"
                                                class="flex justify-between text-slate-500 dark:text-slate-400"
                                            >
                                                <span>
                                                    VAT
                                                    <span v-if="Number(order.vat_percentage || 0) > 0">
                                                        ({{ order.vat_percentage }}%)
                                                    </span>
                                                </span>

                                                <span class="font-medium text-slate-700 dark:text-slate-300">
                                                    + {{ money(order.vat) }}
                                                </span>
                                            </div>

                                        </div>


                                        <!-- Total -->
                                        <div
                                            class="flex justify-between items-baseline
                                                border-t border-slate-100 dark:border-slate-800
                                                mt-4 pt-4"
                                        >
                                            <span class="text-sm font-bold text-slate-900 dark:text-white">
                                                Total Payable
                                            </span>

                                            <span
                                                class="text-xl font-mono font-bold
                                                    text-indigo-600 dark:text-indigo-400"
                                            >
                                                {{ money(order.payable_amount) }}
                                            </span>
                                        </div>


                                        <!-- Paid / Due -->
                                        <div
                                            class="mt-4 pt-4
                                                border-t border-dashed
                                                border-slate-200 dark:border-slate-700
                                                space-y-2"
                                        >

                                            <div class="flex justify-between items-center text-sm">

                                                <span class="text-slate-500 dark:text-slate-400">
                                                    <i class="fa-solid fa-circle-check text-emerald-500 mr-1.5"></i>
                                                    Paid
                                                </span>

                                                <span class="font-mono font-semibold text-emerald-600 dark:text-emerald-400">
                                                    {{ money(paidAmount) }}
                                                </span>

                                            </div>


                                            <div class="flex justify-between items-center text-sm">

                                                <span class="text-slate-500 dark:text-slate-400">
                                                    <i class="fa-solid fa-hourglass-half mr-1.5 text-slate-400"></i>
                                                    Due
                                                </span>

                                                <span
                                                    class="font-mono font-semibold"
                                                    :class="Number(order.due_amount) > 0
                                                        ? 'text-amber-600 dark:text-amber-400'
                                                        : 'text-emerald-600 dark:text-emerald-400'"
                                                >
                                                    {{ money(order.due_amount) }}
                                                </span>

                                            </div>


                                            <span
                                                v-if="Number(order.due_amount) > 0"
                                                class="inline-flex items-center gap-1
                                                    text-[11px] font-semibold
                                                    text-amber-700 dark:text-amber-400
                                                    bg-amber-50 dark:bg-amber-500/10
                                                    px-2.5 py-1 rounded-full mt-1"
                                            >
                                                <i class="fa-solid fa-triangle-exclamation"></i>
                                                Partially Paid
                                            </span>

                                            <span
                                                v-else
                                                class="inline-flex items-center gap-1
                                                    text-[11px] font-semibold
                                                    text-emerald-700 dark:text-emerald-400
                                                    bg-emerald-50 dark:bg-emerald-500/10
                                                    px-2.5 py-1 rounded-full mt-1"
                                            >
                                                <i class="fa-solid fa-circle-check"></i>
                                                Fully Paid
                                            </span>

                                        </div>

                                        <!-- Device Details -->
                                        <details
                                            class="mt-4 pt-4 border-t border-dashed border-slate-200 dark:border-slate-700 text-xs group"
                                        >
                                            <summary
                                                class="text-slate-400 dark:text-slate-500 cursor-pointer select-none list-none flex items-center justify-between"
                                            >
                                                <span>
                                                    <i class="fa-solid fa-circle-info mr-1"></i>
                                                    Device details
                                                </span>

                                                <i
                                                    class="fa-solid fa-chevron-down text-[10px] transition-transform group-open:rotate-180"
                                                ></i>
                                            </summary>

                                            <div class="mt-3 space-y-3">

                                                <!-- Browser -->
                                                <div class="grid grid-cols-2 gap-3 items-center">
                                                    <span class="text-slate-400 dark:text-slate-500">
                                                        Browser
                                                    </span>

                                                    <span
                                                        class="text-right font-medium text-slate-700 dark:text-slate-300 truncate"
                                                        :title="order.user_agent || 'N/A'"
                                                    >
                                                        {{ browserName }}
                                                    </span>
                                                </div>


                                                <!-- Operating System -->
                                                <div class="grid grid-cols-2 gap-3 items-center">
                                                    <span class="text-slate-400 dark:text-slate-500">
                                                        Operating System
                                                    </span>

                                                    <span
                                                        class="text-right font-medium text-slate-700 dark:text-slate-300 truncate"
                                                    >
                                                        {{ operatingSystem }}
                                                    </span>
                                                </div>


                                                <!-- IP Address -->
                                                <div class="grid grid-cols-2 gap-3 items-center">
                                                    <span class="text-slate-400 dark:text-slate-500">
                                                        IP Address
                                                    </span>

                                                    <div class="flex justify-end">
                                                        <button
                                                            type="button"
                                                            @click="toggleIpAddress"
                                                            class="inline-flex items-center gap-2
                                                                px-2.5 py-1.5
                                                                rounded-lg
                                                                bg-slate-100 dark:bg-slate-800
                                                                hover:bg-slate-200 dark:hover:bg-slate-700
                                                                text-slate-600 dark:text-slate-300
                                                                font-medium
                                                                transition"
                                                        >
                                                            <i
                                                                :class="showIpAddress
                                                                    ? 'fa-solid fa-eye-slash'
                                                                    : 'fa-solid fa-eye'"
                                                                class="text-[11px]"
                                                            ></i>

                                                            <span v-if="showIpAddress">
                                                                {{ order.ip_address || 'N/A' }}
                                                            </span>

                                                            <span v-else>
                                                                •••••••••••
                                                            </span>
                                                        </button>
                                                    </div>
                                                </div>


                                                <!-- User Agent -->
                                                <div class="grid grid-cols-2 gap-3 items-start">
                                                    <span class="text-slate-400 dark:text-slate-500">
                                                        User Agent
                                                    </span>

                                                    <span
                                                        class="text-right font-medium text-slate-700 dark:text-slate-300 break-all leading-relaxed"
                                                        :title="order.user_agent || 'N/A'"
                                                    >
                                                        {{ order.user_agent || 'N/A' }}
                                                    </span>
                                                </div>


                                                <!-- Created -->
                                                <div class="grid grid-cols-2 gap-3 items-center">
                                                    <span class="text-slate-400 dark:text-slate-500">
                                                        Created
                                                    </span>

                                                    <span
                                                        class="text-right font-medium text-slate-700 dark:text-slate-300"
                                                    >
                                                        {{ formatDateTime(order.created_at) }}
                                                    </span>
                                                </div>

                                            </div>
                                        </details>

                                    </div>

                                    <!-- Payment History -->
                                    <div class="px-6 pb-6">

                                        <!-- Payment History -->
                                        <div v-if="orderPayments.length" class="pt-4 border-t border-slate-100 dark:border-slate-800">
                                            <div class="flex items-center justify-between mb-3">
                                                <h4 class="text-xs font-bold uppercase tracking-wide text-slate-400 dark:text-slate-500">
                                                    Payment History
                                                </h4>

                                                <span class="text-[11px] font-semibold text-slate-400">
                                                    {{ orderPayments.length }} transaction<span v-if="orderPayments.length !== 1">s</span>
                                                </span>
                                            </div>

                                            <div class="space-y-2">
                                                <div
                                                    v-for="(payment, index) in orderPayments"
                                                    :key="payment.id || index"
                                                    class="rounded-xl bg-slate-50 dark:bg-slate-800/60 border border-slate-100 dark:border-slate-700 overflow-hidden"
                                                >
                                                    <!-- Payment Summary -->
                                                    <button
                                                        type="button"
                                                        @click="togglePaymentDetails(payment.id || index)"
                                                        class="w-full flex items-center justify-between gap-3 px-3 py-3 text-left hover:bg-slate-100 dark:hover:bg-slate-800 transition"
                                                    >
                                                        <div class="min-w-0 flex-1">
                                                            <div class="flex items-center gap-2 flex-wrap">
                                                                <span class="text-xs font-bold text-slate-800 dark:text-slate-200 font-mono">
                                                                    {{ payment.payment_number || payment.receipt_no || `Payment #${index + 1}` }}
                                                                </span>

                                                                <span
                                                                    v-if="payment.payment_type"
                                                                    class="text-[10px] px-1.5 py-0.5 rounded bg-blue-100 dark:bg-blue-500/10 text-blue-700 dark:text-blue-400 font-semibold uppercase"
                                                                >
                                                                    {{ payment.payment_type }}
                                                                </span>

                                                                <span
                                                                    class="text-[10px] px-1.5 py-0.5 rounded bg-emerald-100 dark:bg-emerald-500/10 text-emerald-700 dark:text-emerald-400 font-semibold uppercase"
                                                                >
                                                                    {{ payment.payment_method || 'N/A' }}
                                                                </span>
                                                            </div>

                                                            <p class="text-[11px] text-slate-400 mt-1">
                                                                {{ payment.user?.name || 'N/A' }}

                                                                <span v-if="payment.paid_at">
                                                                    · {{ formatDateTime(payment.paid_at) }}
                                                                </span>
                                                            </p>
                                                        </div>

                                                        <div class="shrink-0 flex items-center gap-2">
                                                            <span class="font-mono font-bold text-emerald-600 dark:text-emerald-400">
                                                                + {{ money(payment.amount) }}
                                                            </span>

                                                            <svg
                                                                class="w-4 h-4 text-slate-400 transition-transform duration-200"
                                                                :class="{ 'rotate-180': expandedPaymentId === (payment.id || index) }"
                                                                fill="none"
                                                                stroke="currentColor"
                                                                viewBox="0 0 24 24"
                                                            >
                                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                                                            </svg>
                                                        </div>
                                                    </button>

                                                    <!-- Payment Details (expanded) -->
                                                    <div v-if="expandedPaymentId === (payment.id || index)" class="px-3 pb-3">
                                                        <div class="pt-3 border-t border-slate-200 dark:border-slate-700">
                                                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">

                                                                <div v-if="payment.id" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Payment ID</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">#{{ payment.id }}</p>
                                                                </div>

                                                                <div v-if="payment.payment_number" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Payment Number</p>
                                                                    <p class="text-xs font-mono font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ payment.payment_number }}</p>
                                                                </div>

                                                                <div v-if="payment.receipt_no" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Receipt No</p>
                                                                    <p class="text-xs font-mono font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ payment.receipt_no }}</p>
                                                                </div>

                                                                <div v-if="payment.payment_type" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Payment Type</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5 capitalize">{{ payment.payment_type }}</p>
                                                                </div>

                                                                <div v-if="payment.payment_method" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Payment Method</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ payment.payment_method }}</p>
                                                                </div>

                                                                <div v-if="Number(payment.discount || 0) > 0" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Discount</p>
                                                                    <p class="text-xs font-bold text-emerald-600 dark:text-emerald-400 mt-0.5">{{ money(payment.discount) }}</p>
                                                                </div>

                                                                <div class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Amount</p>
                                                                    <p class="text-xs font-bold text-emerald-600 dark:text-emerald-400 mt-0.5">{{ money(payment.amount) }}</p>
                                                                </div>

                                                                <div v-if="payment.currency" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Currency</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5 uppercase">{{ payment.currency }}</p>
                                                                </div>

                                                                <div v-if="payment.paid_at" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Paid At</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ formatDateTime(payment.paid_at) }}</p>
                                                                </div>

                                                                <div v-if="payment.verified_at" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Verified At</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ formatDateTime(payment.verified_at) }}</p>
                                                                </div>

                                                                <div v-if="payment.supplier" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Supplier</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ payment.supplier?.name || 'N/A' }}</p>
                                                                </div>

                                                                <div v-if="payment.user" class="p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-semibold text-slate-400">Received By</p>
                                                                    <p class="text-xs font-semibold text-slate-700 dark:text-slate-300 mt-0.5">{{ payment.user?.name || 'N/A' }}</p>
                                                                </div>

                                                            </div>

                                                            <!-- Remarks -->
                                                            <div v-if="payment.remarks" class="mt-2 p-2.5 rounded-lg bg-white dark:bg-slate-900 border border-slate-100 dark:border-slate-700">
                                                                <p class="text-[10px] uppercase font-semibold text-slate-400">Remarks</p>
                                                                <p class="text-xs text-slate-700 dark:text-slate-300 mt-1 whitespace-pre-wrap">{{ payment.remarks }}</p>
                                                            </div>

                                                            <!-- Technical Information -->
                                                            <div v-if="payment.ip_address || payment.user_agent" class="mt-2">
                                                                <div class="px-2.5 py-2 rounded-lg bg-slate-100 dark:bg-slate-900/70 border border-slate-200 dark:border-slate-700">
                                                                    <p class="text-[10px] uppercase font-bold tracking-wide text-slate-400 mb-2">
                                                                        Technical Information
                                                                    </p>

                                                                    <div class="space-y-2">
                                                                        <div v-if="payment.ip_address">
                                                                            <p class="text-[10px] font-semibold text-slate-400">IP Address</p>
                                                                            <p class="text-xs font-mono text-slate-700 dark:text-slate-300 break-all">{{ payment.ip_address }}</p>
                                                                        </div>

                                                                        <div v-if="payment.user_agent">
                                                                            <p class="text-[10px] font-semibold text-slate-400">User Agent</p>
                                                                            <p class="text-[11px] font-mono text-slate-600 dark:text-slate-400 break-all leading-relaxed">{{ payment.user_agent }}</p>
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


                                <!-- Order Items -->
                                <div class="space-y-4">

                                    <!-- Section Header -->
                                    <div class="flex items-center justify-between mb-5">
                                        <div class="flex items-center gap-3">
                                            <div
                                                class="flex h-10 w-10 items-center justify-center rounded-xl
                                                    bg-indigo-50 dark:bg-indigo-500/10
                                                    text-indigo-600 dark:text-indigo-400"
                                            >
                                                <i class="fa-solid fa-bag-shopping"></i>
                                            </div>

                                            <div>
                                                <h3 class="text-lg font-bold text-slate-900 dark:text-white leading-tight">
                                                    Order Items
                                                </h3>

                                                <p class="text-xs text-slate-400 dark:text-slate-500 mt-0.5">
                                                    Products included in this order
                                                </p>
                                            </div>
                                        </div>

                                        <span
                                            class="inline-flex items-center justify-center min-w-8 h-8 px-2
                                                rounded-lg bg-slate-100 dark:bg-slate-800
                                                text-xs font-bold text-slate-600 dark:text-slate-300"
                                        >
                                            {{ cartItems?.length || 0 }}
                                        </span>
                                    </div>


                                    <!-- Table Card -->
                                    <div
                                        class="bg-white dark:bg-slate-900
                                            rounded-2xl
                                            border border-slate-200 dark:border-slate-800
                                            shadow-sm
                                            overflow-hidden"
                                    >

                                        <!-- Desktop / Tablet Table -->
                                        <div class="hidden sm:block overflow-x-auto">
                                            <table class="w-full text-sm">

                                                <thead>
                                                    <tr
                                                        class="bg-slate-50 dark:bg-slate-800/60
                                                            border-b border-slate-200 dark:border-slate-800"
                                                    >
                                                        <th class="text-left font-bold text-[11px] uppercase tracking-wider text-slate-500 dark:text-slate-400 px-5 py-3.5 w-[42%]">
                                                            Product
                                                        </th>
                                                        <th class="text-left font-bold text-[11px] uppercase tracking-wider text-slate-500 dark:text-slate-400 px-5 py-3.5">
                                                            Unit Price
                                                        </th>
                                                        <th class="text-center font-bold text-[11px] uppercase tracking-wider text-slate-500 dark:text-slate-400 px-5 py-3.5">
                                                            Quantity
                                                        </th>
                                                        <th class="text-right font-bold text-[11px] uppercase tracking-wider text-slate-500 dark:text-slate-400 px-5 py-3.5">
                                                            Subtotal
                                                        </th>
                                                    </tr>
                                                </thead>

                                                <tbody class="divide-y divide-slate-100 dark:divide-slate-800">

                                                    <tr
                                                        v-for="item in cartItems || []"
                                                        :key="item.id"
                                                        class="group hover:bg-slate-50/80 dark:hover:bg-slate-800/40 transition-colors"
                                                    >

                                                        <!-- Product -->
                                                        <td class="px-5 py-4">
                                                            <div class="flex items-center gap-3.5 min-w-0">

                                                                <div
                                                                    class="relative w-14 h-14 shrink-0
                                                                        bg-slate-100 dark:bg-slate-800
                                                                        rounded-lg overflow-hidden
                                                                        border border-slate-200 dark:border-slate-700"
                                                                >
                                                                    <img
                                                                        :src="getProductImage(item)"
                                                                        :alt="item.product?.name || 'Product'"
                                                                        class="h-full w-full object-cover
                                                                            group-hover:scale-105
                                                                            transition-transform duration-500"
                                                                        @error="$event.target.src = defaultProductImage"
                                                                    />
                                                                </div>

                                                                <div class="min-w-0">
                                                                    <p
                                                                        class="font-semibold text-slate-900 dark:text-white truncate"
                                                                        :title="item.product?.name || item.product_name || 'Product'"
                                                                    >
                                                                        {{ item.product?.name || item.product_name || 'Product' }}
                                                                    </p>

                                                                    <p
                                                                        v-if="item.product?.sku || item.sku"
                                                                        class="text-xs text-slate-400 dark:text-slate-500 mt-0.5"
                                                                    >
                                                                        SKU: {{ item.product?.sku || item.sku }}
                                                                    </p>
                                                                </div>

                                                            </div>
                                                        </td>

                                                        <!-- Unit Price -->
                                                        <td class="px-5 py-4 text-slate-600 dark:text-slate-300 font-medium whitespace-nowrap">
                                                            {{ money(item.unit_price ?? item.price ?? 0) }}
                                                        </td>

                                                        <!-- Quantity -->
                                                        <td class="px-5 py-4 text-center">
                                                            <span
                                                                class="inline-flex items-center justify-center
                                                                    min-w-8 h-7 px-2
                                                                    rounded-lg
                                                                    bg-indigo-50 dark:bg-indigo-500/10
                                                                    text-indigo-600 dark:text-indigo-400
                                                                    text-xs font-bold"
                                                            >
                                                                ×{{ item.quantity }}
                                                            </span>
                                                        </td>

                                                        <!-- Subtotal -->
                                                        <td class="px-5 py-4 text-right whitespace-nowrap">
                                                            <span class="font-bold text-slate-900 dark:text-white">
                                                                {{
                                                                    money(
                                                                        item.subtotal ??
                                                                        (
                                                                            Number(item.unit_price ?? item.price ?? 0) *
                                                                            Number(item.quantity ?? 0)
                                                                        )
                                                                    )
                                                                }}
                                                            </span>
                                                        </td>

                                                    </tr>

                                                </tbody>

                                                <!-- Footer total -->
                                                <tfoot v-if="cartItems && cartItems.length">
                                                    <tr class="bg-slate-50/60 dark:bg-slate-800/40 border-t border-slate-200 dark:border-slate-800">
                                                        <td colspan="3" class="px-5 py-3.5 text-right text-xs font-bold uppercase tracking-wider text-slate-500 dark:text-slate-400">
                                                            Items Total
                                                        </td>
                                                        <td class="px-5 py-3.5 text-right">
                                                            <span class="text-base font-black text-indigo-600 dark:text-indigo-400">
                                                                {{
                                                                    money(
                                                                        (cartItems || []).reduce((sum, item) =>
                                                                            sum + Number(
                                                                                item.subtotal ??
                                                                                (Number(item.unit_price ?? item.price ?? 0) * Number(item.quantity ?? 0))
                                                                            ), 0
                                                                        )
                                                                    )
                                                                }}
                                                            </span>
                                                        </td>
                                                    </tr>
                                                </tfoot>

                                            </table>
                                        </div>


                                        <!-- Mobile Stacked Rows -->
                                        <div class="sm:hidden divide-y divide-slate-100 dark:divide-slate-800">

                                            <div
                                                v-for="item in cartItems || []"
                                                :key="item.id"
                                                class="p-4 flex gap-3.5"
                                            >

                                                <div
                                                    class="relative w-16 h-16 shrink-0
                                                        bg-slate-100 dark:bg-slate-800
                                                        rounded-lg overflow-hidden
                                                        border border-slate-200 dark:border-slate-700"
                                                >
                                                    <img
                                                        :src="getProductImage(item)"
                                                        :alt="item.product?.name || 'Product'"
                                                        class="h-full w-full object-cover"
                                                        @error="$event.target.src = defaultProductImage"
                                                    />
                                                </div>

                                                <div class="flex-1 min-w-0">

                                                    <p class="font-semibold text-sm text-slate-900 dark:text-white truncate">
                                                        {{ item.product?.name || item.product_name || 'Product' }}
                                                    </p>

                                                    <p
                                                        v-if="item.product?.sku || item.sku"
                                                        class="text-xs text-slate-400 dark:text-slate-500 mt-0.5"
                                                    >
                                                        SKU: {{ item.product?.sku || item.sku }}
                                                    </p>

                                                    <div class="flex items-center justify-between mt-2.5">
                                                        <div class="flex items-center gap-2 text-xs text-slate-500 dark:text-slate-400">
                                                            <span>{{ money(item.unit_price ?? item.price ?? 0) }}</span>
                                                            <span class="text-slate-300 dark:text-slate-700">×</span>
                                                            <span class="font-semibold text-indigo-600 dark:text-indigo-400">{{ item.quantity }}</span>
                                                        </div>

                                                        <span class="font-bold text-sm text-slate-900 dark:text-white">
                                                            {{
                                                                money(
                                                                    item.subtotal ??
                                                                    (Number(item.unit_price ?? item.price ?? 0) * Number(item.quantity ?? 0))
                                                                )
                                                            }}
                                                        </span>
                                                    </div>

                                                </div>

                                            </div>

                                        </div>


                                        <!-- Empty State -->
                                        <div
                                            v-if="!cartItems || cartItems.length === 0"
                                            class="px-6 py-12 text-center"
                                        >
                                            <div
                                                class="mx-auto mb-4
                                                    flex h-14 w-14
                                                    items-center justify-center
                                                    rounded-2xl
                                                    bg-slate-100 dark:bg-slate-800
                                                    text-slate-400 dark:text-slate-500"
                                            >
                                                <i class="fa-solid fa-box-open text-xl"></i>
                                            </div>

                                            <h4 class="text-sm font-bold text-slate-700 dark:text-slate-300">
                                                No order items found
                                            </h4>

                                            <p class="mt-1 text-xs text-slate-400 dark:text-slate-500">
                                                There are no products associated with this order.
                                            </p>
                                        </div>

                                    </div>

                                </div>

                            </div>


                            <!-- RIGHT -->
                            <div class="space-y-6">

                                <div class="sticky top-16">

                                    <!-- Supplier -->
                                    <div
                                        class="bg-white dark:bg-slate-900 rounded-2xl border border-slate-200 dark:border-slate-800 shadow-sm overflow-hidden"
                                    >
                                        <div
                                            class="px-6 py-4 border-b border-slate-100 dark:border-slate-800 bg-slate-50/50 dark:bg-slate-800/50"
                                        >
                                            <h3 class="font-bold text-slate-900 dark:text-white">
                                                Supplier Details
                                            </h3>
                                        </div>

                                        <div class="p-6">

                                            <!-- Supplier Avatar -->
                                            <div class="text-center mb-6">

                                                <div class="relative inline-flex mb-4">
                                                    <div
                                                        class="h-20 w-20 rounded-2xl
                                                            bg-gradient-to-tr from-indigo-600 to-purple-500
                                                            flex items-center justify-center
                                                            text-white text-2xl font-bold
                                                            shadow-lg shadow-indigo-500/30"
                                                    >
                                                        {{ supplierInitials }}
                                                    </div>

                                                    <div
                                                        class="absolute -bottom-2 -right-2
                                                            bg-green-500
                                                            border-4 border-white dark:border-slate-900
                                                            h-6 w-6 rounded-full"
                                                        title="Active"
                                                    ></div>
                                                </div>

                                                <h4 class="text-lg font-bold text-slate-900 dark:text-white">
                                                    {{ order.supplier_name || order.supplier?.name || 'Unknown Supplier' }}
                                                </h4>

                                                <p class="text-sm text-slate-500 mt-1">
                                                    {{ order.supplier_phone || order.supplier?.phone || 'No phone number' }}
                                                </p>

                                            </div>


                                            <!-- Supplier Information -->
                                            <div
                                                class="rounded-2xl
                                                    border border-slate-200/70 dark:border-slate-800
                                                    bg-white dark:bg-slate-900
                                                    divide-y divide-slate-100 dark:divide-slate-800
                                                    shadow-sm"
                                            >

                                                <!-- Supplier Name -->
                                                <div class="flex items-center gap-3 p-4">

                                                    <span
                                                        class="flex items-center justify-center
                                                            w-8 h-8 rounded-lg
                                                            bg-slate-50 dark:bg-slate-800
                                                            ring-1 ring-slate-200/80 dark:ring-slate-700
                                                            text-slate-500 dark:text-slate-400 shrink-0"
                                                    >
                                                        <i class="fa-regular fa-user text-[13px]"></i>
                                                    </span>

                                                    <div class="min-w-0">
                                                        <p class="text-[10px] text-slate-400 uppercase font-semibold tracking-wider">
                                                            Supplier Name
                                                        </p>

                                                        <p class="text-sm font-semibold text-slate-800 dark:text-slate-100 truncate">
                                                            {{ order.supplier_name || order.supplier?.name || 'N/A' }}
                                                        </p>
                                                    </div>

                                                </div>


                                                <!-- Phone -->
                                                <div class="flex items-center gap-3 p-4">

                                                    <span
                                                        class="flex items-center justify-center
                                                            w-8 h-8 rounded-lg
                                                            bg-slate-50 dark:bg-slate-800
                                                            ring-1 ring-slate-200/80 dark:ring-slate-700
                                                            text-slate-500 dark:text-slate-400 shrink-0"
                                                    >
                                                        <i class="fa-solid fa-phone text-[13px]"></i>
                                                    </span>

                                                    <div class="min-w-0">
                                                        <p class="text-[10px] text-slate-400 uppercase font-semibold tracking-wider">
                                                            Phone Number
                                                        </p>

                                                        <p class="text-sm font-semibold text-slate-800 dark:text-slate-100 truncate">
                                                            {{ order.supplier_phone || order.supplier?.phone || 'N/A' }}
                                                        </p>
                                                    </div>

                                                </div>


                                                <!-- Supplier ID -->
                                                <div class="flex items-center gap-3 p-4">

                                                    <span
                                                        class="flex items-center justify-center
                                                            w-8 h-8 rounded-lg
                                                            bg-slate-50 dark:bg-slate-800
                                                            ring-1 ring-slate-200/80 dark:ring-slate-700
                                                            text-slate-500 dark:text-slate-400 shrink-0"
                                                    >
                                                        <i class="fa-solid fa-truck-field text-[13px]"></i>
                                                    </span>

                                                    <div class="min-w-0">
                                                        <p class="text-[10px] text-slate-400 uppercase font-semibold tracking-wider">
                                                            Supplier ID
                                                        </p>

                                                        <p class="text-sm font-semibold text-slate-800 dark:text-slate-100">
                                                            {{ order.supplier_id || 'N/A' }}
                                                        </p>
                                                    </div>

                                                </div>

                                            </div>


                                            <!-- Processed By -->
                                            <div
                                                v-if="order.user"
                                                class="mt-5 p-4 rounded-xl
                                                    bg-slate-50 dark:bg-slate-800/50
                                                    border border-slate-100 dark:border-slate-800"
                                            >
                                                <p class="text-[10px] text-slate-400 uppercase font-semibold tracking-wider mb-1">
                                                    Processed By
                                                </p>

                                                <p class="text-sm font-bold text-slate-800 dark:text-slate-100">
                                                    {{ order.user.name }}
                                                </p>

                                                <p class="text-xs text-slate-500 mt-0.5">
                                                    {{ order.user.user_id }}
                                                </p>
                                            </div>


                                            <!-- Remarks -->
                                            <div
                                                v-if="order.remarks"
                                                class="mt-5"
                                            >
                                                <div
                                                    class="w-full flex items-start gap-3.5
                                                        pl-4 border-l-2
                                                        border-amber-300 dark:border-amber-600"
                                                >
                                                    <div class="min-w-0 flex-1">

                                                        <p
                                                            class="text-[10.5px]
                                                                text-amber-600 dark:text-amber-500
                                                                uppercase font-bold tracking-wider mb-1"
                                                        >
                                                            <i class="fa-regular fa-comment-dots mr-1"></i>
                                                            Note
                                                        </p>

                                                        <p class="text-sm text-slate-600 dark:text-slate-400 leading-relaxed italic">
                                                            {{ order.remarks }}
                                                        </p>

                                                    </div>
                                                </div>
                                            </div>

                                        </div>
                                    </div>


                                    <!-- Quick Note -->
                                    <div
                                        class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl p-5 mt-4 shadow-sm"
                                    >

                                        <div class="flex items-start gap-3">

                                            <div
                                                class="flex-shrink-0 w-9 h-9 rounded-full bg-blue-50 dark:bg-blue-900/30 flex items-center justify-center"
                                            >
                                                <i class="fa-solid fa-circle-info text-blue-600 dark:text-blue-400 text-sm"></i>
                                            </div>

                                            <div>

                                                <h4
                                                    class="font-semibold text-sm text-slate-800 dark:text-slate-100 mb-1"
                                                >
                                                    Quick Note
                                                </h4>

                                                <p
                                                    class="text-slate-500 dark:text-slate-400 text-xs leading-relaxed"
                                                >
                                                    This order was processed by
                                                    <strong>{{ order.user?.name || 'System' }}</strong>.
                                                </p>

                                            </div>

                                        </div>

                                    </div>

                                </div>

                            </div>

                        </div>

                    </div>


                </main>
            </div>
        </div>

    </div>
    <FooterSection />
</template>

<script setup>
import { onMounted, ref, computed } from "vue";
import { useRouter, useRoute } from "vue-router";
import api from "../../../../services/api.js";

import Navbar from "../../admin/admin-navbar.vue";
import HeaderSection from "../../admin/admin-header.vue";
import Message from "../../../Message/message.vue";
import FooterSection from "../../../footer.vue";


// =====================================================
// Navigation / UI
// =====================================================

const router = useRouter();
const route = useRoute();

const mobileMenu = ref(false);

function toggleMenu() {
    mobileMenu.value = !mobileMenu.value;
}


// =====================================================
// State
// =====================================================

const loading = ref(false);
const successMsg = ref("");
const errorMsg = ref("");

const order = ref(null);
const cartItems = ref([]);
const payments = ref([]);

const defaultProductImage = "/images/product/default-product.webp";


// =====================================================
// Fetch Purchase Order Details
// =====================================================

async function fetchOrderDetails() {
    loading.value = true;
    errorMsg.value = "";

    try {
        const reg = route.params.reg;

        if (!reg) {
            errorMsg.value = "Invalid purchase order reference.";
            return;
        }

        const res = await api.get(
            `/purchase/order/details/${reg}`
        );

        const data = res.data?.data;

        if (!data?.order) {
            errorMsg.value = "Purchase order not found.";
            order.value = null;
            return;
        }

        order.value = data.order;

        cartItems.value = data.cartItems || [];

        payments.value = data.payments || [];

    } catch (err) {

        errorMsg.value =
            err.response?.data?.message ||
            err.message ||
            "Something went wrong while fetching purchase order.";

    } finally {
        loading.value = false;
    }
}


// =====================================================
// Product Image
// =====================================================

const getProductImage = (item) => {

    const images = item?.product?.images || [];

    if (!images.length) {
        return defaultProductImage;
    }

    // Primary image first
    const image =
        images.find(img => Number(img.is_primary) === 1) ||
        images[0];

    if (!image?.image_path) {
        return defaultProductImage;
    }

    const path = image.image_path;

    // Already full URL
    if (
        path.startsWith("http://") ||
        path.startsWith("https://")
    ) {
        return path;
    }

    // Already starts with /
    if (path.startsWith("/")) {
        return path;
    }

    return `/storage/${path}`;
};


// =====================================================
// Money Formatter
// =====================================================

const money = (value) => {

    const amount = Number(value || 0);

    return `৳ ${amount.toLocaleString("en-BD", {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
    })}`;
};


// =====================================================
// Date Formatter
// =====================================================

const formatDate = (date) => {

    if (!date) {
        return "N/A";
    }

    return new Date(date).toLocaleDateString("en-BD", {
        year: "numeric",
        month: "short",
        day: "2-digit",
    });
};


// =====================================================
// Date + Time Formatter
// =====================================================

const formatDateTime = (date) => {

    if (!date) {
        return "N/A";
    }

    return new Date(date).toLocaleString("en-BD", {
        year: "numeric",
        month: "short",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
    });
};


// =====================================================
// Status Formatter
// =====================================================

const formatStatus = (status) => {

    if (!status) {
        return "N/A";
    }

    return String(status)
        .replace(/_/g, " ")
        .replace(/\b\w/g, char => char.toUpperCase());
};


// =====================================================
// Paid Amount
// =====================================================

const paidAmount = computed(() => {

    return Number(
        order.value?.paid_amount || 0
    );
});


// =====================================================
// Supplier Initials
// =====================================================

const supplierInitials = computed(() => {

    const name =
        order.value?.supplier_name ||
        order.value?.supplier?.name ||
        "Supplier";

    const parts = name
        .trim()
        .split(/\s+/)
        .filter(Boolean);

    if (!parts.length) {
        return "SU";
    }

    if (parts.length === 1) {
        return parts[0]
            .substring(0, 2)
            .toUpperCase();
    }

    return (
        parts[0].charAt(0) +
        parts[parts.length - 1].charAt(0)
    ).toUpperCase();
});


// =====================================================
// Browser
// =====================================================

const browserName = computed(() => {
    const ua = order.value?.user_agent || "";

    if (!ua) {
        return "Unknown";
    }

    // Edge must be checked before Chrome
    if (/Edg/i.test(ua)) {
        return "Microsoft Edge";
    }

    // Opera must be checked before Chrome
    if (/Opera|OPR/i.test(ua)) {
        return "Opera";
    }

    if (/Firefox/i.test(ua)) {
        return "Firefox";
    }

    if (/Chrome/i.test(ua)) {
        return "Google Chrome";
    }

    if (/Safari/i.test(ua)) {
        return "Safari";
    }

    return "Unknown Browser";
});


// =====================================================
// Operating System
// =====================================================

const operatingSystem = computed(() => {
    const ua = order.value?.user_agent || "";

    if (!ua) {
        return "Unknown";
    }

    // Chrome OS
    if (/CrOS/i.test(ua)) {
        return "Chrome OS";
    }

    // Windows
    if (/Windows NT 10.0/i.test(ua)) {
        return "Windows 10/11";
    }

    if (/Windows NT 6.3/i.test(ua)) {
        return "Windows 8.1";
    }

    if (/Windows NT 6.2/i.test(ua)) {
        return "Windows 8";
    }

    if (/Windows NT 6.1/i.test(ua)) {
        return "Windows 7";
    }

    // Android
    if (/Android/i.test(ua)) {
        const match = ua.match(/Android\s([0-9.]+)/i);

        return match
            ? `Android ${match[1]}`
            : "Android";
    }

    // iOS
    if (/iPhone|iPad|iPod/i.test(ua)) {
        return "iOS";
    }

    // macOS
    if (/Mac OS X/i.test(ua)) {
        return "macOS";
    }

    // Linux
    if (/Linux/i.test(ua)) {
        return "Linux";
    }

    return "Unknown";
});

const showIpAddress = ref(false);

const toggleIpAddress = () => {
    showIpAddress.value = !showIpAddress.value;
};

// =====================================================
// Status Style
// =====================================================

const statusClass = (status) => {

    switch (
        String(status || "").toLowerCase()
    ) {

        case "completed":

            return `
                bg-emerald-50
                dark:bg-emerald-500/10
                text-emerald-700
                dark:text-emerald-400
                border-emerald-200
                dark:border-emerald-500/20
            `;


        case "pending":

            return `
                bg-amber-50
                dark:bg-amber-500/10
                text-amber-700
                dark:text-amber-400
                border-amber-200
                dark:border-amber-500/20
            `;


        case "unpaid":

            return `
                bg-red-50
                dark:bg-red-500/10
                text-red-700
                dark:text-red-400
                border-red-200
                dark:border-red-500/20
            `;


        case "partially_paid":

            return `
                bg-orange-50
                dark:bg-orange-500/10
                text-orange-700
                dark:text-orange-400
                border-orange-200
                dark:border-orange-500/20
            `;


        case "returned":

            return `
                bg-purple-50
                dark:bg-purple-500/10
                text-purple-700
                dark:text-purple-400
                border-purple-200
                dark:border-purple-500/20
            `;


        default:

            return `
                bg-slate-100
                dark:bg-slate-800
                text-slate-600
                dark:text-slate-300
                border-slate-200
                dark:border-slate-700
            `;
    }
};


// =====================================================
// Payment History (expand/collapse)
// =====================================================

const expandedPaymentId = ref(null);

function togglePaymentDetails(id) {
    expandedPaymentId.value =
        expandedPaymentId.value === id ? null : id;
}

const orderPayments = computed(() => payments.value || []);


// =====================================================
// Product Details
// =====================================================

function ProductDetails(item) {

    const slug = item?.product?.slug;

    if (!slug) {
        return;
    }

    router.push(
        `/product-details/${slug}`
    );
}


// =====================================================
// Print Purchase Order
// =====================================================

function printOrder(purchaseOrder) {

    if (!purchaseOrder?.reg) {
        errorMsg.value =
            "Invalid purchase order reference.";

        return;
    }

    const win =
        window.open(
            "about:blank",
            "_blank"
        );

    if (!win) {

        alert(
            "Popup blocked! Please allow popups."
        );

        return;
    }

    win.location.href =
        `/admin/purchase/order/invoice-print/${purchaseOrder.reg}`;
}


// =====================================================
// Download Invoice
// =====================================================

const invoiceDownloading = ref(false);

async function downloadInvoice() {

    if (!order.value?.reg) {
        errorMsg.value =
            "Invalid purchase order reference.";

        return;
    }

    invoiceDownloading.value = true;

    try {

        const res = await api.get(
            `/purchase/order/${order.value.reg}/invoice`,
            {
                responseType: "blob"
            }
        );

        const blob =
            new Blob([res.data], {
                type:
                    res.headers?.["content-type"] ||
                    "application/pdf"
            });

        const url =
            window.URL.createObjectURL(blob);

        const link =
            document.createElement("a");

        link.href = url;

        link.setAttribute(
            "download",
            `purchase-order-${order.value.order_number || order.value.reg}.pdf`
        );

        document.body.appendChild(link);

        link.click();

        link.remove();

        window.URL.revokeObjectURL(url);

    } catch (err) {

        errorMsg.value =
            err.response?.data?.message ||
            err.message ||
            "Could not download the purchase order invoice.";

    } finally {

        invoiceDownloading.value = false;

    }
}


// =====================================================
// Dark / Light Mode
// =====================================================

const isDark = ref(false);

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


// =====================================================
// Mounted
// =====================================================

onMounted(() => {

    fetchOrderDetails();


    // ESC → close mobile menu
    window.addEventListener(
        "keydown",
        (e) => {

            if (e.key === "Escape") {
                mobileMenu.value = false;
            }

        }
    );


    // Theme
    const saved =
        localStorage.getItem("theme");

    if (saved === "dark") {

        applyTheme(true);

    } else if (saved === "light") {

        applyTheme(false);

    } else {

        applyTheme(
            window.matchMedia(
                "(prefers-color-scheme: dark)"
            ).matches
        );

    }

});
</script>

<style>

</style>