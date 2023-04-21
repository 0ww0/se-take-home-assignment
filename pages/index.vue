<script setup>
    const state = reactive({
        orders: [],
        completedOrders: [],
        bots: [],
        botCount: 0,
        botNum: 0,
        orderNum: 0,
        processingOrder: false,
    });

    const addOrder = (type) => {
        state.orderNum++;
        const order = {
            orderNumber: state.orderNum,
            type,
            time: 10,
            botId: null,
            status: "Pending",
        };
        if (type === "VIP") {
            let vipIndex = state.orders.findIndex((o) => o.type !== "VIP");
            if (vipIndex === -1) vipIndex = state.orders.length;
            state.orders.splice(vipIndex, 0, order);
        } else {
            state.orders.push(order);
        }
    };

    const addBot = () => {
        state.botNum++;
        const bot = { 
            id: state.botNum, 
            status: "Idle" 
        }
        state.bots.push(bot);
        state.botCount++;
    };

    const removeBot = () => {
        const inUseBot = state.bots.find((bot) => bot.status === "In Use");
        const idleBot = state.bots.find((bot) => bot.status === "Idle");
        if (inUseBot) {
            const botOrder = state.orders.find(
                (order) => order.botId === inUseBot.id && order.status === "Pending"
            );
            if (botOrder) {
                clearInterval(botOrder.interval);
                botOrder.botId = null;
                botOrder.time = 10;
                state.processingOrder = false;
            }
            state.bots = state.bots.filter((bot) => bot.id !== inUseBot.id);
            state.botCount--;
        } else if (idleBot) {
            state.bots = state.bots.filter((bot) => bot.id !== idleBot.id);
            state.botCount--;
        }
    };

    watchEffect(() => {
        if (state.processingOrder) return;
        const ordersToProcess = state.orders.filter((order) => order.botId === null);
        const availableBots = state.bots.filter((bot) => bot.status === "Idle");
        for (const order of ordersToProcess) {
            const availableBot = availableBots.find(
                (bot) =>
                    !state.orders.some(
                        (o) =>
                            o.botId === bot.id &&
                            o.status === "Pending" &&
                            o.orderNumber !== order.orderNumber
                    )
            );
            if (!availableBot) continue; 
            const previouslyAssignedBot = state.bots.find(
                (bot) => bot.id === order.botId && bot.status === "Idle"
            );
            if (previouslyAssignedBot) availableBot = previouslyAssignedBot;
            order.botId = availableBot.id;
            availableBot.status = "In Use";
            state.processingOrder = true; 
            startProcessingOrder(order.botId, order);
            break; 
        }
    });

    // watchEffect(() => {
    //     const idleBots = state.bots.filter(bot => bot.status === "Idle");
    //     const ordersToProcess = state.orders.filter(order => order.botId === null);
        
    //     ordersToProcess.forEach(order => {
    //         const availableBot = idleBots.find(bot => {
    //         const assignedOrders = 
    //             state.orders.filter(o => o.botId === bot.id && o.status === "Pending");
    //             return assignedOrders.length === 0;
    //         });
            
    //         if (!availableBot) return;
            
    //         order.botId = availableBot.id;
    //         availableBot.status = "In Use";
    //         state.processingOrder = true; 
    //         startProcessingOrder(order.botId, order);
    //     });
    // });

    const startProcessingOrder = (botId, order) => {
        order.interval = setInterval(() => {
            order.time--;
            if (order.time === 0) {
                clearInterval(order.interval);
                order.botId = null;
                order.status = "Completed";
                const bot = state.bots.find((bot) => bot.id === botId);
                bot.status = "Idle";
                state.completedOrders.push(order);
                state.orders = state.orders.filter((o) => o.status === "Pending");
                state.processingOrder = false;
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
                            <span v-if="order.time < 10">(Processing in {{ order.time }}s)</span>
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
