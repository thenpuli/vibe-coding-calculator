<template>
  <div class="calculator">
    <div class="display">
      <div class="result">{{ displayValue }}</div>
    </div>
    <div class="buttons">
      <div class="row">
        <button 
          class="button function" 
          :class="{ active: currentOperation === 'clear' }"
          @click="clear"
        >
          {{ allClear ? 'AC' : 'C' }}
        </button>
        <button 
          class="button function" 
          @click="toggleSign"
        >
          +/-
        </button>
        <button 
          class="button function" 
          @click="percentage"
        >
          %
        </button>
        <button 
          class="button operation" 
          :class="{ active: currentOperation === '/' }"
          @click="setOperation('/')"
        >
          ÷
        </button>
      </div>
      <div class="row">
        <button class="button number" @click="appendNumber('7')">7</button>
        <button class="button number" @click="appendNumber('8')">8</button>
        <button class="button number" @click="appendNumber('9')">9</button>
        <button 
          class="button operation" 
          :class="{ active: currentOperation === '*' }"
          @click="setOperation('*')"
        >
          ×
        </button>
      </div>
      <div class="row">
        <button class="button number" @click="appendNumber('4')">4</button>
        <button class="button number" @click="appendNumber('5')">5</button>
        <button class="button number" @click="appendNumber('6')">6</button>
        <button 
          class="button operation" 
          :class="{ active: currentOperation === '-' }"
          @click="setOperation('-')"
        >
          -
        </button>
      </div>
      <div class="row">
        <button class="button number" @click="appendNumber('1')">1</button>
        <button class="button number" @click="appendNumber('2')">2</button>
        <button class="button number" @click="appendNumber('3')">3</button>
        <button 
          class="button operation" 
          :class="{ active: currentOperation === '+' }"
          @click="setOperation('+')"
        >
          +
        </button>
      </div>
      <div class="row">
        <button class="button number zero" @click="appendNumber('0')">0</button>
        <button class="button number" @click="appendDecimal">.</button>
        <button 
          class="button operation" 
          @click="calculate"
        >
          =
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

export default defineComponent({
  name: 'App',
  setup() {
    const currentValue = ref('0');
    const previousValue = ref<string | null>(null);
    const currentOperation = ref<string | null>(null);
    const shouldResetDisplay = ref(false);
    const allClear = ref(true);

    const displayValue = computed(() => {
      // Format the display value to match iPhone calculator
      // (e.g., add commas for thousands)
      const value = currentValue.value;
      if (value === 'Error') return value;
      
      // Handle decimal numbers
      if (value.includes('.')) {
        const [intPart, decPart] = value.split('.');
        const formattedInt = Number(intPart).toLocaleString();
        return `${formattedInt}.${decPart}`;
      }
      
      return Number(value).toLocaleString();
    });

    const clear = () => {
      if (allClear.value) {
        currentValue.value = '0';
        previousValue.value = null;
        currentOperation.value = null;
      } else {
        currentValue.value = '0';
        allClear.value = true;
      }
    };

    const appendNumber = (number: string) => {
      allClear.value = false;
      
      if (shouldResetDisplay.value) {
        currentValue.value = number;
        shouldResetDisplay.value = false;
      } else {
        // Replace initial 0 with the number, unless it's a decimal
        if (currentValue.value === '0' && number !== '0') {
          currentValue.value = number;
        } else if (currentValue.value !== '0') {
          // Limit the length to match iPhone calculator (typically 9 digits)
          if (currentValue.value.replace(/[^0-9]/g, '').length < 9) {
            currentValue.value += number;
          }
        }
      }
    };

    const appendDecimal = () => {
      allClear.value = false;
      
      if (shouldResetDisplay.value) {
        currentValue.value = '0.';
        shouldResetDisplay.value = false;
      } else if (!currentValue.value.includes('.')) {
        currentValue.value += '.';
      }
    };

    const toggleSign = () => {
      if (currentValue.value !== '0' && currentValue.value !== 'Error') {
        currentValue.value = currentValue.value.startsWith('-') 
          ? currentValue.value.substring(1) 
          : `-${currentValue.value}`;
      }
    };

    const percentage = () => {
      if (currentValue.value !== 'Error') {
        const value = parseFloat(currentValue.value);
        currentValue.value = (value / 100).toString();
      }
    };

    const setOperation = (operation: string) => {
      if (currentValue.value === 'Error') return;
      
      if (currentOperation.value && previousValue.value && !shouldResetDisplay.value) {
        calculate();
      }
      
      previousValue.value = currentValue.value;
      currentOperation.value = operation;
      shouldResetDisplay.value = true;
    };

    const calculate = () => {
      if (!previousValue.value || !currentOperation.value || currentValue.value === 'Error') {
        return;
      }
      
      try {
        const prev = parseFloat(previousValue.value);
        const current = parseFloat(currentValue.value);
        let result = 0;
        
        switch (currentOperation.value) {
          case '+':
            result = prev + current;
            break;
          case '-':
            result = prev - current;
            break;
          case '*':
            result = prev * current;
            break;
          case '/':
            if (current === 0) {
              currentValue.value = 'Error';
              previousValue.value = null;
              currentOperation.value = null;
              shouldResetDisplay.value = true;
              return;
            }
            result = prev / current;
            break;
        }
        
        // Handle potential floating point issues
        const resultStr = result.toString();
        if (resultStr.includes('.') && resultStr.split('.')[1].length > 10) {
          currentValue.value = result.toFixed(10).replace(/\.?0+$/, '');
        } else {
          currentValue.value = resultStr;
        }
        
        previousValue.value = null;
        currentOperation.value = null;
        shouldResetDisplay.value = true;
      } catch (e) {
        currentValue.value = 'Error';
        previousValue.value = null;
        currentOperation.value = null;
        shouldResetDisplay.value = true;
      }
    };

    return {
      displayValue,
      currentOperation,
      allClear,
      clear,
      appendNumber,
      appendDecimal,
      toggleSign,
      percentage,
      setOperation,
      calculate
    };
  }
});
</script>

<style scoped>
.calculator {
  display: flex;
  flex-direction: column;
  height: 100%;
  background-color: #000;
}

.display {
  flex: 1;
  display: flex;
  justify-content: flex-end;
  align-items: flex-end;
  padding: 0 20px;
  color: white;
  font-size: 80px;
  font-weight: 300;
  text-align: right;
  overflow: hidden;
}

.result {
  width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.buttons {
  display: flex;
  flex-direction: column;
  padding-bottom: 20px;
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
  padding: 0 10px;
}

.button {
  width: 75px;
  height: 75px;
  border-radius: 50%;
  border: none;
  font-size: 30px;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: filter 0.2s;
}

.button:active {
  filter: brightness(130%);
}

.function {
  background-color: #a5a5a5;
  color: #000;
}

.number {
  background-color: #333;
  color: white;
}

.operation {
  background-color: #ff9f0a;
  color: white;
}

.operation.active {
  background-color: white;
  color: #ff9f0a;
}

.zero {
  width: 162px;
  border-radius: 37.5px;
  justify-content: flex-start;
  padding-left: 28px;
}
</style>
