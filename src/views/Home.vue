<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from '../components/Tasks'
import AddTask from '../components/AddTask'
export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
      alertedTasks: new Set(),
    }
  },
  methods: {
    checkTaskAlerts() {
      const now = new Date()
      this.tasks.forEach((task) => {
        if (!task.day || !task.time) return
        if (this.alertedTasks.has(task.id)) return
        const time24 = this.to24Hour(task.time)
        const taskDate = new Date(task.day + 'T' + time24)
        const diff = (now - taskDate) / 1000
        if (diff >= 0 && diff < 60) {
          this.alertedTasks.add(task.id)
          alert(`Reminder: "${task.text}" is scheduled for ${task.day} at ${task.time}`)
        }
      })
    },
    to24Hour(timeStr) {
      const [time, modifier] = timeStr.split(' ')
      let [h, m] = time.split(':').map(Number)
      if (modifier === 'PM' && h !== 12) h += 12
      if (modifier === 'AM' && h === 12) h = 0
      return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}`
    },
    async addTask(task) {
      const res = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(task),
      })

      const data = await res.json()

      this.tasks = [...this.tasks, data]
    },
    async deleteTask(id) {
      if (confirm('Are you sure?')) {
        const res = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        })

        res.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert('Error deleting task')
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id)
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder }

      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updTask),
      })

      const data = await res.json()

      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      )
    },
    async fetchTasks() {
      const res = await fetch('api/tasks')

      const data = await res.json()

      return data
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`)

      const data = await res.json()

      return data
    },
  },
  async created() {
    this.tasks = await this.fetchTasks()
    this.checkTaskAlerts()
    this._alertInterval = setInterval(() => this.checkTaskAlerts(), 30000)
  },
  beforeUnmount() {
    clearInterval(this._alertInterval)
  },
}
</script>
