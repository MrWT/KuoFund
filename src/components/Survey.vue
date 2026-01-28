<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Survey mounted.");
        init();
    });

    let appState = ref("ERROR");
    let appMessage = ref("");
    let members = reactive([]);
    let locations = reactive([ 
        "土城郭家", 
        "山上區-809親子農場" 
    ]);
    let location_descList = reactive([
        "#",
        "https://anity0404.pixnet.net/blog/posts/8223469887",
    ]);
    let surveys = reactive([]);
    let selMemCode = ref("");
    let selLocation = ref("");

    // 初始化 component
    function init(){
        console.log("Survey.init");
        console.log("Survey.props.title", props.title);
        console.log("Survey.props.account", props.account);

        fetchInitData();
        checkValue();
    }
    // 取得初始資料
    function fetchInitData(){
        // 取得成員
        let fetchPromise_members = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        // 取得 Survey 資料
        let fetchPromise_surveys = fetchData({
            api: "get_surveys",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_members, fetchPromise_surveys]).then((values) => {
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
            }

            // survey
            {
                surveys.splice(0, surveys.length);
                values[1].forEach((surveyObj, survey_i) => {
                    surveys.push({ member_code: surveyObj["member_code"], location: surveyObj["location"] });
                });
                console.log("surveys=", surveys);
            }
        });
    }
    // 確認地點
    function saveLocation(){
        console.log("saveLocation.selMemCode=" + selMemCode.value);
        console.log("saveLocation.selLocation=" + selLocation.value);

        let api_name = "new_survey";
        surveys.forEach((surveyObj, survey_i) => {
            if(surveyObj["member_code"] === selMemCode.value){
                api_name = "update_survey";
            }
        });
        console.log("api_name=" + api_name);

        // 新增/更新 survey
        let editPromise = fetchData({
            api: api_name,
            data: {
                member_code: selMemCode.value,
                location: selLocation.value,
            }
        }, "KUO-FUNDS");
        Promise.all([editPromise]).then((editValues) => {
            console.log(api_name + ".editValues=", editValues);

        });
    }
    // 檢查數值
    function checkValue(){
        let message = "";
    
        if(!selMemCode.value){
            appState.value = "ERROR";
            message += (message.length > 0 ? " / " : "") + "請選擇成員姓名!";
        }
        if(!selLocation.value){
            appState.value = "ERROR";
            message += (message.length > 0 ? " / " : "") + "請選擇地點!";
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

        selLocation.value = "";
        surveys.forEach((surveyObj, survey_i) => {
            if( surveyObj["member_code"] === selMemCode.value ){
                selLocation.value = surveyObj["location"];
            }
        });
        // 檢查數值
        checkValue();
    });
    watch(selLocation, (newValue, oldValue) => {
        // 檢查數值
        checkValue();
    });
</script>

<template>

    <div class="w-1/1 h-1/1 flex flex-col gap-3">
        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-2/3 text-center text-2xl underline">
                2026 中秋聚餐地點問卷調查
            </div>
        </div>

        <div class="w-1/1 flex flex-row items-center gap-3">
            <span class="flex-none p-1 px-2 bg-gray-200">姓名:</span>
            <select class="flex-1 border border-1 p-1 text-lg rounded-xl" v-model="selMemCode">
                <option v-for="(memObj, mem_i) in members" :value="memObj.code_name">{{ memObj.name }}</option>
            </select>
        </div>

        <div class="w-1/1 flex flex-row items-start gap-3">
            <span class="flex-none p-1 px-2 bg-gray-200">偏好地點:</span>
            <div class="flex-1 flex flex-col">
                <div v-for="(loc, loc_i) in locations" class="w-1/1">
                    <input type="radio" :value="loc" v-model="selLocation" />
                    <a :href="location_descList[loc_i]" target="_blank" class="px-5"
                        :class="{'underline cursor-pointer': location_descList[loc_i] !== '#'}">{{ loc }}</a>
                </div>
            </div>
        </div>

        <div v-if="appState === 'SUCCESS'" class="w-1/1 flex flex-col place-items-center border-0 border-t-1">
            <div class="w-2/3 flex flex-row items-start justify-center gap-3 mt-2">
                <a class="w-1/2 rounded-xl p-2 flex flex-row justify-center items-center cursor-pointer bg-green-500/50 text-gray-900 hover:bg-green-500/100" @click="saveLocation">
                    <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m4 6 2 2 4-4m4-8v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                    </svg>
                    <span>
                        確認
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
