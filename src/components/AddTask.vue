<template>
  <form @submit="onSubmit" class="add-form">
    <div class="form-control">
      <label>Task</label>
      <input type="text" v-model="text" name="text" placeholder="Add Task" />
    </div>
    <div class="form-control">
      <label>Day</label>
      <input type="date" v-model="day" name="day" />
    </div>
    <div class="form-control">
      <label>Time</label>
      <div class="time-row">
        <select v-model.number="hour" class="time-select">
          <option v-for="h in 12" :key="h" :value="h">{{ h.toString().padStart(2, '0') }}</option>
        </select>
        <span class="time-sep">:</span>
        <select v-model.number="minute" class="time-select">
          <option v-for="m in [0,5,10,15,20,25,30,35,40,45,50,55]" :key="m" :value="m">{{ m.toString().padStart(2, '0') }}</option>
        </select>
        <button type="button" class="ampm-toggle" @click="toggleAmPm">{{ ampm }}</button>
      </div>
    </div>
    <div class="form-control form-control-check">
      <label>Set Reminder</label>
      <input type="checkbox" v-model="reminder" name="reminder" />
    </div>

    <input type="submit" value="Save Task" class="btn btn-block" />
  </form>
</template>

<script>
export default {
  name: 'AddTask',
  data() {
    return {
      text: '',
      day: '',
      hour: 12,
      minute: 0,
      ampm: 'AM',
      reminder: false,
    }
  },
  computed: {
    time() {
      const h = this.hour.toString().padStart(2, '0')
      const m = this.minute.toString().padStart(2, '0')
      return `${h}:${m} ${this.ampm}`
    },
  },
  methods: {
    toggleAmPm() {
      this.ampm = this.ampm === 'AM' ? 'PM' : 'AM'
    },
    onSubmit(e) {
      e.preventDefault()

      if (!this.text) {
        alert('Please add a task')
        return
      }

      const newTask = {
        text: this.text,
        day: this.day,
        time: this.time,
        reminder: this.reminder,
      }

      this.$emit('add-task', newTask)

      this.text = ''
      this.day = ''
      this.hour = 12
      this.minute = 0
      this.ampm = 'AM'
      this.reminder = false
    },
  },
}
</script>

<style scoped>
.add-form {
  margin-bottom: 40px;
}

.form-control {
  margin: 20px 0;
}

.form-control label {
  display: block;
}

.form-control input {
  width: 100%;
  height: 40px;
  margin: 5px;
  padding: 3px 7px;
  font-size: 17px;
}

.form-control-check {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.form-control-check label {
  flex: 1;
}

.form-control-check input {
  flex: 2;
  height: 20px;
}

.time-row {
  display: flex;
  align-items: center;
  gap: 4px;
  margin: 5px;
}

.time-select {
  width: 70px;
  height: 40px;
  font-size: 17px;
  padding: 3px 7px;
}

.time-sep {
  font-size: 20px;
  font-weight: bold;
}

.ampm-toggle {
  height: 40px;
  width: 60px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #f4f4f4;
}

.ampm-toggle:hover {
  background: #e0e0e0;
}
</style>
