<script setup>
    const state = reactive({
        orders: [],
        completedOrders: [],
        bots: [],
        botCount: 0,
    });

    watchEffect(() => {
        const ordersToProcess = state.orders.filter((order) => order.botId === null);
        const availableBots = state.bots.filter((bot) => bot.status === "Idle");

        for (const order of ordersToProcess) {
            // Find an available bot that is not already assigned to a pending order
            const availableBot = availableBots.find(
            (bot) =>
                !state.orders.some(
                (o) =>
                    o.botId === bot.id &&
                    o.status === "Pending" &&
                    o.orderNumber !== order.orderNumber // ignore the current order
                )
            );

            if (availableBot) {
                // Assign the available bot to the order
                order.botId = availableBot.id;
                availableBot.status = "In Use";
                processOrder(order.botId, order);
            }
        }
    });

    let orderCount = 0;

    const addOrder = (type) => {
        orderCount++;
        const orderNumber = orderCount;
        const order = { orderNumber, type, time: 10, botId: null, status: "Pending" };
        if (type === "VIP") {
            state.orders.unshift(order); // insert VIP order at the beginning
        } else {
            state.orders.push(order);
        }
    };

    const filterCompletedOrders = () => {
        state.orders = state.orders.filter((order) => order.status === "Pending");
    };

    let bots = 0

    const addBot = () => {
        bots++;
        const id = bots;
        const bot = { id, status: "Idle" };
        state.bots.push(bot);
        state.botCount++;
    };

    const removeBot = () => {
        const availableBot = state.bots.find(bot => bot.status === "Idle");

        if (availableBot) {
            const botOrder = state.orders.find(
                order => order.botId === availableBot.id && order.status === "Pending"
            );

            if (botOrder) {
                botOrder.botId = null;
                botOrder.time = 10;
                clearInterval(botOrder.interval);
            }

            state.bots = state.bots.filter(bot => bot.id !== availableBot.id);
            state.botCount--;
        }
    };

    const processOrder = (botId, order) => {
        order.interval = setInterval(() => {
            order.time--;
            if (order.time === 0) {
                clearInterval(order.interval);
                order.botId = null;
                order.status = "Completed";
                const bot = state.bots.find((bot) => bot.id === botId);
                bot.status = "Idle";
                state.completedOrders.push(order);
                filterCompletedOrders();
            }
        }, 1000);
    };
</script>

<template>
<div class="container mx-auto p-6">
    <div class="divider">
        <div class="flex items-center pb-3">
            <div class="border-t flex-grow"></div>
            <div class="mx-4 text-gray-500">Order</div>
            <div class="border-t flex-grow"></div>
        </div>
        <div class="flex flex-wrap justify-evenly h-full">
            <div class="w-1/2 h-full p-5">
                <h1 class="uppercase text-2xl pb-4">Pending</h1>
                <div class="w-full h-80 p-4 bg-slate-100 rounded-xl">
                    <ul class="w-full h-full overflow-auto">
                        <li v-for="order in state.orders" 
                            :key="order.orderNumber">
                            Order No # {{ order.orderNumber }} ({{ order.type }})
                            <span v-if="order.time !== 10">(Processing in {{ order.time }}s)</span>
                        </li>
                    </ul>
                </div>
            </div>
            <div class="w-1/2 h-full p-5">
                <h1 class="uppercase text-2xl pb-4">Completed</h1>
                <div class="w-full h-80 p-4 bg-slate-100 rounded-xl">
                    <ul class="w-full h-full overflow-auto">
                        <li v-for="order in state.completedOrders" 
                            :key="order.orderNumber">
                            Order No # {{ order.orderNumber }} ({{ order.type }})
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="button-group flex justify-center">
            <Button 
                tag="button" 
                text="Add Normal Orders"
                class="mr-2"
                @click="addOrder('Normal')"
            />
            <Button 
                tag="button" 
                text="Add VIP Orders"
                class="ml-2"
                @click="addOrder('VIP')"
            />
        </div>
        <div class="border-b flex-grow pt-3"></div>
    </div>
    <div class="divider">
        <div class="flex items-center pb-3">
            <div class="border-t flex-grow"></div>
            <div class="mx-4 text-gray-500">Bots</div>
            <div class="border-t flex-grow"></div>
        </div>
        <div class="button-group flex justify-center">
            <Button 
                tag="button" 
                text="+ Bots"
                class="mr-2"
                @click="addBot()"
            />
            <Button 
                tag="button" 
                text="- Bots"
                class="ml-2"
                @click="removeBot()"
                :disabled="state.bots.length === 0"
            />
        </div>
        <div class="w-1/2 h-full p-5">
            <h1 class="uppercase text-2xl pb-4">Bot Available</h1>
            <div class="w-full h-48 p-4 bg-slate-100 rounded-xl">
                <ul class="w-full h-full overflow-auto">
                    <li v-for="bot in state.bots" 
                        :key="bot.id">
                        Bot # {{ bot.id }} ({{ bot.status }})
                    </li>
                </ul>
            </div>
        </div>
        <div class="border-b flex-grow pt-3"></div>
    </div>
</div>
</template>
