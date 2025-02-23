<script>
import CalculatorButton from "@/components/CalculatorButton.vue";

export default {
  name: "Calculator",
  components: {
    CalculatorButton,
  },
  data() {
    return {
      numberData: ["7", "8", "9", "4", "5", "6", "1", "2", "3", "0"],
      operatorData: ["/", "*", "-", "+"],
      actionData: [
        { name: "Reset", style: "reset" },
        { name: "Del", style: "del" },
      ],
      calStore: [], // 儲存使用者輸入的 字串數字 和 運算符
      viewOperator: "", // 儲存使用者輸入的 運算符
      calNumber: "0", // 使用者輸入的 字串數字
      ishandleOperator: false, // 使用者是否點擊過運算符
      state: "inputNumber", // 點擊 等於 後針對各狀態的操作
      lastOperand: "", //儲存最後的 數字字串
      lastOperator: "", //儲存最後的 運算符
      isResult: false, // 是否點擊 等於
      isOverflow: false, // 是否超出限制
    };
  },
  methods: {
    calResult(expression) {
      // console.log("測試1", this.calStore);
      if (/\/\s*0+(\D|$)/.test(expression)) {
        // console.log("測試2");
        this.isOverflow = true;
        this.calNumber = "不能除以 0";
        return;
      }
      const temp = new Function(`return ${expression}`)().toString();

      if (temp.length > 10) {
        this.isOverflow = true;
        this.calNumber = "超出計算最大限制";
        return;
      } else {
        this.state = "inputEqual";
        this.lastOperand = this.calStore.at(-1);
        this.lastOperator = this.calStore.at(-2);
        this.calStore = [temp]; // 清空並存入計算結果
        this.calNumber = temp;
      }
    },
    handleReset() {
      this.calStore = [];
      this.calNumber = "0";
      this.viewOperator = "";
      this.ishandleOperator = false;
      this.state = "inputNumber";
      this.lastOperand = "";
      this.lastOperator = "";
      this.isResult = false;
    },
    handleDel() {
      if (this.state === "inputEqual") {
        this.handleReset();
      }
      // 如果當前數字長度為1，或是"0"，則設為"0"
      if (this.calNumber.length === 1 || this.calNumber === "0") {
        this.calNumber = "0";
      } else {
        // 移除最後一個字符
        this.calNumber = this.calNumber.slice(0, -1);
      }
    },
    handleNumber(num) {
      // 限制最大長度為 10
      this.handleOverFlow();
      if (this.isResult) {
        this.handleReset();
      }
      this.state = "inputNumber";
      this.isResult = false;
      if (this.calNumber.length >= 10) return;
      this.calNumber += num;
      // 正則移除開頭多餘的 0（但允許單獨 "0"）
      this.calNumber = this.calNumber.replace(/^(-?)0+(?=\d)/, "$1");
      this.ishandleOperator = false;
    },
    handleOperator(arithmetic) {
      this.handleOverFlow();
      this.state = "inputOperator";
      this.isResult = false;
      this.viewOperator = arithmetic;
      if (this.ishandleOperator) {
        this.calStore[this.calStore.length - 1] = arithmetic;
        return;
      }

      if (this.state === "inputEqual") {
        // 如果是計算後的狀態，則繼續運算
        this.calStore = [this.calStore.at(-1), arithmetic];
      } else {
        if (isNaN(this.calStore.at(-1))) {
          this.calStore.push(this.calNumber);
        }
        this.calStore.push(arithmetic);
      }

      this.ishandleOperator = true;
      this.calNumber = "0";
      // console.log(this.calStore);
    },
    handleEqual() {
      this.handleOverFlow();
      this.isResult = true;
      this.viewOperator = "";
      switch (this.state) {
        case "inputNumber":
          if (!this.calStore.length || isNaN(this.calStore.at(-1))) {
            this.calStore.push(this.calNumber);
          }
          break;

        case "inputOperator":
          if (this.operatorData.includes(this.calStore.at(-1))) {
            this.calStore.pop();
          }
          break;

        case "inputEqual":
          // 重複計算
          // console.log("哈囉");
          if (this.lastOperator && this.lastOperand) {
            this.calStore.push(this.lastOperator);
            this.calStore.push(this.lastOperand);
            // console.log("重複計算", this.calStore);
          }
          break;
      }
      // console.log(this.calStore);
      this.calResult(this.calStore.join(""));
    },
    handleAction(action) {
      if (action === "Reset") {
        this.handleReset();
      }
      if (action === "Del") {
        this.handleDel();
      }
    },
    handleOverFlow() {
      if (this.isOverflow) {
        this.handleReset();
        this.isOverflow = false;
      }
    },
  },
};
</script>

<template>
  <div class="container">
    <div class="calculator">
      <div class="output-view">
        <p class="output-text">
          {{ calNumber }}
        </p>
        <span class="operator-view">{{ viewOperator }}</span>
      </div>
      <div class="buttons">
        <div>
          <div class="action">
            <CalculatorButton v-for="item in actionData" :key="item.name" :class="item.style" :handleButton="handleAction" :name="item.name">
              {{ item.name }}
            </CalculatorButton>
          </div>
          <div class="numbers">
            <template v-for="item in numberData" :key="item">
              <CalculatorButton v-if="item === '0'" class="number zero-number" :handleButton="handleNumber" :name="item">
                {{ item }}
              </CalculatorButton>
              <CalculatorButton v-else class="number" :handleButton="handleNumber" :name="item">
                {{ item }}
              </CalculatorButton>
            </template>
          </div>
        </div>
        <div class="operators">
          <CalculatorButton v-for="item in operatorData" :key="item" class="operator" :handleButton="handleOperator" :name="item">
            {{ item }}
          </CalculatorButton>
          <CalculatorButton class="equal" :handleButton="handleEqual" :name="'='">=</CalculatorButton>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
