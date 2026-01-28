<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Register mounted.");
        init();
    });

    let appState = ref("ERROR");
    let appMessage = ref("");
    let members = reactive([]);
    let registers = reactive([]);
    let is_register = ref(false);
    let selMemCode = ref("");
    let selPerpleCount = ref("0");

    // 初始化 component
    function init(){
        console.log("Register.init");
        console.log("Register.props.title", props.title);
        console.log("Register.props.account", props.account);

        fetchInitData();
        checkValue();
    }
    // 取得初始資料
    function fetchInitData(){
        let tmpMemCode = selMemCode.value;
        selMemCode.value = "";
        selPerpleCount.value = 0;
        is_register.value = false;

        // 取得成員
        let fetchPromise_members = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        // 取得 Register 資料
        let fetchPromise_registers = fetchData({
            api: "get_registers",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_members, fetchPromise_registers]).then((values) => {
            console.log("fetchInitData.values=", values);

            // 成員
            {
                members.splice(0, members.length);
                values[0].forEach((memObj, mem_i) => {
                    members.push({ code_name: memObj["code_name"], name: memObj["name"] });
                });
                members.sort((x, y) => {
                    if(x["code_name"] > y["code_name"]) return 1;
                    if(x["code_name"] < y["code_name"]) return -1;
                    return 0;
                });

                selMemCode.value = tmpMemCode;
            }

            // register
            {
                registers.splice(0, registers.length);
                values[1].forEach((reObj, re_i) => {
                    registers.push({ member_code: reObj["member_code"], people_count: reObj["people_count"] });
                });
                console.log("registers=", registers);
            }
        });
    }
    // 確認登記
    function saveRegister(){
        console.log("cancelRegister.selMemCode=" + selMemCode.value);

        // 新增/更新 register
        let editPromise = fetchData({
            api: "update_register",
            data: {
                activity_name: "2026淨灘",
                member_code: selMemCode.value,
                people_count: selPerpleCount.value,
            }
        }, "KUO-FUNDS");
        Promise.all([editPromise]).then((editValues) => {
            console.log("update_register.editValues=", editValues);
            fetchInitData();
        });
    }
    // 取消登記
    function cancelRegister(){
        console.log("cancelRegister.selMemCode=" + selMemCode.value);

        // 取消 register
        let editPromise = fetchData({
            api: "cancel_register",
            data: {
                member_code: selMemCode.value,
            }
        }, "KUO-FUNDS");
        Promise.all([editPromise]).then((editValues) => {
            console.log("cancel_register.editValues=", editValues);
            fetchInitData();
        });
    }
    // 檢查數值
    function checkValue(){
        let message = "";
    
        if(!selMemCode.value){
            appState.value = "ERROR";
            message += (message.length > 0 ? " / " : "") + "請選擇成員姓名!";
        }
        if(!selPerpleCount.value){
            appState.value = "ERROR";
            message += (message.length > 0 ? " / " : "") + "請填寫參與人數!";
        }else{
            if(parseInt( selPerpleCount.value ) === 0){
                appState.value = "ERROR";
                message += (message.length > 0 ? " / " : "") + "請填寫參與人數!";
            }
        }

        if(message.length > 0){
            appState.value = "ERROR";
        }else{
            appState.value = "SUCCESS";
        }
        appMessage.value = message;
    }

    // 監聽
    watch(selMemCode, (newValue, oldValue) => {
        console.log("watch.selMemCode=" + selMemCode.value);

        selPerpleCount.value = 0;
        is_register.value = false;
        if(selMemCode.value){
            registers.forEach((reObj, re_i) => {
                if(reObj["member_code"] === selMemCode.value){
                    is_register.value = true;

                    selPerpleCount.value = reObj["people_count"];
                }
            });
        }

        // 檢查數值
        checkValue();
    });
    watch(selPerpleCount, (newValue, oldValue) => {
        // 檢查數值
        checkValue();
    });
</script>

<template>

    <div class="w-1/1 h-1/1 flex flex-col gap-3">
        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-2/3 text-center text-2xl underline">
                2026 淨灘活動
            </div>
        </div>
        <div class="w-1/1 flex flex-row justify-center">
            <div class="w-2/3 text-center text-xl">
                <a class="cursor-pointer underline px-2" href="https://www.facebook.com/p/%E5%8F%B0%E5%8D%97%E6%B7%A8%E9%81%8A%E8%B6%A3-100080277601505/?locale=zh_TW" target="_blank">
                    主辦單位: 台南淨遊趣
                </a>        
            </div>
        </div>

        <div class="w-1/1 flex flex-row items-center gap-3">
            <span class="flex-none p-1 px-2 bg-gray-200">姓名:</span>
            <select class="flex-1 border border-1 p-1 text-lg rounded-xl" v-model="selMemCode">
                <option v-for="(memObj, mem_i) in members" :value="memObj.code_name">{{ memObj.name }}</option>
            </select>
            <div v-if="is_register === true" class="flex-none badge badge-success">已登記</div>
            <div v-if="is_register === false" class="flex-none badge badge-error">未登記</div>
        </div>

        <div class="w-1/1 flex flex-row items-center gap-3">
            <span class="flex-none p-1 px-2 bg-gray-200">人數:</span>
            <input class="flex-1 border border-1 p-1 text-lg rounded-xl" type="number" min="0" step="1" v-model="selPerpleCount" />
        </div>

        <div v-if="appState === 'SUCCESS'" class="w-1/1 flex flex-col place-items-center border-0 border-t-1">
            <div class="w-2/3 flex flex-row items-start justify-center gap-3 mt-2">
                <a class="w-1/2 rounded-xl p-2 flex flex-row justify-center items-center cursor-pointer bg-rose-500/50 text-gray-900 hover:bg-rose-500/100" @click="cancelRegister">
                    <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m4 6 2 2 4-4m4-8v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                    </svg>
                    <span>
                        取消登記
                    </span>
                </a>
                <a class="w-1/2 rounded-xl p-2 flex flex-row justify-center items-center cursor-pointer bg-green-500/50 text-gray-900 hover:bg-green-500/100" @click="saveRegister">
                    <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m4 6 2 2 4-4m4-8v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                    </svg>
                    <span>
                        確認登記
                    </span>
                </a>
            </div>
        </div>

        <div v-if="appState === 'ERROR'" class="w-1/1 flex flex-col place-items-center">
            <div class="w-1/1 text-center rounded-xl text-xl p-2 bg-rose-300">
                {{ appMessage }}
            </div>
        </div>
    </div>

</template>

<style scoped>

</style>
