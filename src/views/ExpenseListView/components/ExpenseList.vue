<template >
    <div class="div">
      <div class="div-2">
        <div class="div-3">
            <div class="div-4">
                <div class="div-5">전체 내역 {{ data.totalCnt }}건</div>
            </div>
            
            <div class="div-7">
                <div class="div-8">
                    <TotalAmount/>
                </div>
            </div>
        </div>
            <div>
                <ul>
                    <li class="list-item-day" v-for="item_day in data.groupedList">
                        <div style="justify-content: space-between; display: flex;">
                            <div class="date">
                                {{ parseInt(item_day[0].date.split("-")[1]) + "월 " + parseInt(item_day[0].date.split("-")[2]) + "일"}}
                            </div>
                            <ExpenseListItemAmount :data="item_day"/>
                        </div> 
                        <li class="list-item-day-detail" v-for="item_detail in item_day">
                            <div class="div-17">
                                <div class="div-18">
                                    <div class="div-19">
                                        <div class="div-20">{{ item_detail.category }}</div>
                                            <div style="display: flex;">
                                                <div class="div-21">{{ item_detail.memo }}</div>
                                                <div class="div-21">{{ item_detail.payment }}</div>
                                            </div>
                                        <div class="div-22">{{ (item_detail.class === "지출") ? "- " : "+ "}}  {{ item_detail.amount.toLocaleString('ko-KR') }}원</div>
                                    </div>
                                </div>
                            </div>
                        </li>
                    </li>
                </ul>
            </div>
        </div>
    </div>

</template>
  
  <script setup>
    import { reactive, onMounted, watch } from 'vue';
    import TotalAmount from '@/components/TotalAmount.vue';
    import axios from 'axios';
    import ExpenseListItemAmount from './ExpenseListItemAmount.vue';
    const data = reactive({list : "", totalCnt : "", groupedList : ""});

    const userId = sessionStorage.getItem("id");

    const props = defineProps({
        search : {
            type : Object,
            required : true
        }
    });

    const updateAccount = (list) => {
        let income = 0;
        let expense = 0;
        let profit = 0; 
        list.forEach(e => {
            if (e.class === "지출") {
            income += e.amount;
            } else {
            expense += e.amount;
            }
        });

        profit = income - expense;

        const accountData = {
            total_income: income,
            total_expand: expense,
            profit: profit
        };
        
        axios.put(`http://localhost:3001/account/${userId}`, accountData)
            .then(res => {
            console.log(res.data);
            })
            .catch(e => {
            console.log(e);
            });
        };


          // 날짜를 키로 잡아서 다시 그룹화 후, 날짜를 기준으로 최신순 정렬하는 메소드
            const groupByDate = (list) => {
                const grouped = list.reduce((acc, item) => {
                    const date = item.date;
                    if (!acc[date]) {
                        acc[date] = [];
                    }
                    acc[date].push(item);
                    return acc;
                    }, {});

                    // 객체의 키를 배열로 변환하여 정렬
                    const sortedKeys = Object.keys(grouped).sort((a, b) => new Date(b) - new Date(a));

                    // 정렬된 키를 기준으로 새로운 객체 생성
                    const sortedGrouped = sortedKeys.reduce((acc, key) => {
                        acc[key] = grouped[key];
                        return acc;
                    }, {});

                    return sortedGrouped;
                };


    const getList = async() => { 
        await axios.get(`http://localhost:3001/transactionDetail?user_id=${userId}`)
        .then(res => {
            data.list = res.data;

            data.groupedList = groupByDate(data.list);

            console.log(data.groupedList);
            data.totalCnt = res.data.length;

            updateAccount(data.list);
            
        })
        .catch(e => {
            console.log(e);
            alert("오류");
        });
    }

    onMounted(() => {
        getList();
    })
            // 부모 컴포넌트로부터 받아온 조건을 필터링
            const filterList = () => {
                const search = props.search;
                const filteredList = data.list.filter(item => {
                    const date = item.date.split('-');
                    
                    const yearCondition = !search.date.year || date[0] === search.date.year;
                    const monthCondition = !search.date.month || date[1] === search.date.month;
                    const dayCondition = !search.date.date || date[2] === search.date.date;
                    const dateCondition = yearCondition && monthCondition && dayCondition;

                    const classCondition = !search.class || item.class === search.class;
                    const categoryCondition = !search.category || item.category === search.category;
                    const paymentCondition = !search.payment || item.payment === search.payment;

                return dateCondition && classCondition && categoryCondition && paymentCondition;
                })

                console.log(filteredList);

                data.groupedList = groupByDate(filteredList);
                data.totalCnt = filteredList.length;
            };

            // props 변경 사항 추적하여 적용
            watch(() => props.search, filterList, { deep: true });

  </script>


  
  
  <style scoped>

  .list-item-day {
    list-style: none;
  }  
  @media (max-width: 991px) {
    .list-item-day {
      max-width: 100%;
    }
  }

  .list-item-day-detail {
    list-style: none;
  }
  @media (max-width: 991px) {
    .list-item-day-detail {
      max-width: 100%;
    }
  }



  .div {
    display: flex;
    padding-bottom: 31px;
    flex-direction: column;
    z-index: auto;
    position: relative;
  }
  .div-2 {
    align-self: center;
    display: flex;
    margin-top: -19px;
    width: 100%;
    max-width: 1064px;
    flex-direction: column;
  }
  @media (max-width: 991px) {
    .div-2 {
      max-width: 100%;
    }
  }
  .div-3 {
    display: flex;
    margin-top: 60px;
    gap: 20px;
    font-weight: 700;
    justify-content: space-between;
  }
  @media (max-width: 991px) {
    .div-3 {
      max-width: 100%;
      flex-wrap: wrap;
      padding-right: 20px;
      margin-top: 40px;
    }
  }
  .div-4 {
    display: flex;
    flex-direction: column;
    color: #434343;
  }
  .div-5 {
    font: xx-large Inter, sans-serif;
  }
  .date {
    margin-top: 23px;
    font: large 500;
  }
  .div-7 {
    align-self: start;
    display: flex;
    flex-direction: column;
    white-space: nowrap;
  }
  @media (max-width: 991px) {
    .div-7 {
      white-space: initial;
    }
  }
  .div-8 {
    display: flex;
    gap: 9px;
    font-size: 11px;
  }
  @media (max-width: 991px) {
    .div-8 {
      white-space: initial;
    }
  }
  .div-9 {
    color: #6293ce;
    font-family: Inter, sans-serif;
    flex-grow: 1;
  }
  .div-10 {
    color: #f66464;
    font-family: Inter, sans-serif;
  }
  
  .div-17 {
    background-color: #f8f8f8;
    display: flex;
    margin-top: 7px;
    flex-direction: column;
    color: #434343;
    font-weight: 700;
  }
  @media (max-width: 991px) {
    .div-17 {
      max-width: 100%;
    }
  }
  .div-18 {
    border-radius: 10px;
    background-color: rgba(248, 248, 248, 1);
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    display: flex;
    width: 100%;
    gap: 20px;
    white-space: nowrap;
    padding: 19px 11px;
    justify-content: space-between;
  }
  @media (max-width: 991px) {
    .div-18 {
      max-width: 100%;
      flex-wrap: wrap;
      white-space: initial;
    }
  }
  .div-19 {
    justify-content: start;
    margin-left: 5%;
    display: flex;
    gap: 10%;
  }
  @media (max-width: 991px) {
    .div-19 {
      white-space: initial;
    }
  }
  .category_traffic {
    border-radius: 11px;
    background-color: #9fd7ff;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: medium Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .category_traffic {
      white-space: initial;
      padding: 0 20px;
    }
  }
  .category_shopping {
    border-radius: 11px;
    background-color: #f9f59e;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: medium Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .category_shopping {
      white-space: initial;
      padding: 0 20px;
    }
  }

  .category_life {
    border-radius: 11px;
    background-color: #99ff89;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: medium Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .category_life {
      white-space: initial;
      padding: 0 20px;
    }
  }

  .category_food {
    border-radius: 11px;
    background-color: #ffade3;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: medium Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .category_food {
      white-space: initial;
      padding: 0 20px;
    }
  }

  .div-21 {
    margin: auto 0 0 10%;
    font: medium Inter, sans-serif;
  }

  .div-22 {
    flex-grow: 1;
    margin: auto 0 0 10%;
    font: medium Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-22 {
      max-width: 100%;
    }
  }
  .div-23 {
    border-radius: 10px;
    background-color: rgba(248, 248, 248, 1);
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    display: flex;
    margin-top: 6px;
    width: 100%;
    gap: 20px;
    padding: 19px 11px;
  }
  @media (max-width: 991px) {
    .div-23 {
      max-width: 100%;
      flex-wrap: wrap;
    }
  }
  .div-24 {
    display: flex;
    gap: 10px;
  }
  .div-25 {
    border-radius: 11px;
    background-color: #f9f59e;
    white-space: nowrap;
    text-align: center;
    justify-content: center;
    padding: 6px 14px;
    font: 11px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-25 {
      white-space: initial;
    }
  }
  .div-26 {
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  .div-27 {
    flex-grow: 1;
    flex-basis: auto;
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-27 {
      max-width: 100%;
    }
  }
  .div-28 {
    border-radius: 10px;
    background-color: rgba(248, 248, 248, 1);
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    display: flex;
    margin-top: 9px;
    width: 100%;
    gap: 20px;
    white-space: nowrap;
    padding: 19px 11px;
  }
  @media (max-width: 991px) {
    .div-28 {
      max-width: 100%;
      flex-wrap: wrap;
      white-space: initial;
    }
  }
  .div-29 {
    display: flex;
    gap: 10px;
  }
  @media (max-width: 991px) {
    .div-29 {
      white-space: initial;
    }
  }
  .div-30 {
    border-radius: 11px;
    background-color: #99ff89;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: 11px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-30 {
      white-space: initial;
      padding: 0 20px;
    }
  }
  .div-31 {
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  .div-32 {
    flex-grow: 1;
    flex-basis: auto;
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-32 {
      max-width: 100%;
    }
  }
  .div-33 {
    color: #434343;
    margin-top: 8px;
    font: 700 12px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-33 {
      max-width: 100%;
    }
  }
  .div-34 {
    border-radius: 10px;
    background-color: rgba(248, 248, 248, 1);
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    display: flex;
    margin-top: 5px;
    width: 100%;
    gap: 20px;
    color: #434343;
    font-weight: 700;
    white-space: nowrap;
    padding: 19px 11px;
  }
  @media (max-width: 991px) {
    .div-34 {
      max-width: 100%;
      flex-wrap: wrap;
      white-space: initial;
    }
  }
  .div-35 {
    display: flex;
    gap: 10px;
  }
  @media (max-width: 991px) {
    .div-35 {
      white-space: initial;
    }
  }
  .div-36 {
    border-radius: 11px;
    background-color: #ffade3;
    align-items: start;
    text-align: center;
    justify-content: center;
    padding: 6px 26px;
    font: 11px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-36 {
      white-space: initial;
      padding: 0 20px;
    }
  }
  .div-37 {
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  .div-38 {
    flex-grow: 1;
    flex-basis: auto;
    margin: auto 0;
    font: 12px Inter, sans-serif;
  }
  @media (max-width: 991px) {
    .div-38 {
      max-width: 100%;
    }
  }
  </style>