<script setup>
import { ref, reactive, computed, onMounted } from 'vue';
import { format } from 'date-fns';
import axios from "axios";
import MailView from './MailView.vue';

const mails = reactive([]);
const modal = ref(null);

onMounted(async () => {
  const response = await axios.get('http://localhost:3000/emails')
  Object.assign(mails, response.data)
})

const readMail = (mail) => {
  mail.read = true
  updateMail(mail)
  toggleModal(mail)
}

const toggleModal = (mail) => {
  if (!modal) {
    modal.value = null;
  } else {
    modal.value = mail
  }
}

const archiveMail = (mail) => {
  mail.archived = true
  updateMail(mail)
}

function updateMail(mail) {
  axios.put(`http://localhost:3000/emails/${mail.id}`, mail)
}

const formatDate = (date) => {
  return format(new Date(date), 'MMM do yyyy')
}

const sortedMails = computed(() => {
  return mails.sort((m1, m2) => {
    return m1.sentAt < m2.sentAt ? 1 : -1
  })
})

const unarchivedMail = computed(() => {
  return sortedMails.value.filter(m => !m.archived)
})

</script>

<template>
  <div class="px-4 mt-20 sm:px-6 lg:px-8">
    <div class="pb-3 border-b-2">
      <h1 class="text-5xl text-center">OsthirMail Inbox</h1>
    </div>
    <div class="flow-root mt-8">
      <div class="-mx-4 -my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="inline-block min-w-full py-2 align-middle sm:px-6 lg:px-8">
          <div class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 sm:rounded-lg">
            <table class="min-w-full divide-y divide-gray-300">
              <tbody class="bg-white divide-y divide-gray-200">
                <tr class="cursor-pointer" v-for="mail in unarchivedMail" :key="mail.id"
                  :class="mail.read ? 'bg-gray-100' : ''" @click="readMail(mail)">
                  <td class="pl-2">
                    <input type="checkbox" />
                  </td>
                  <td class="py-4 pl-4 pr-3 text-sm font-medium text-gray-900 whitespace-nowrap sm:pl-6">{{ mail.from }}
                  </td>
                  <td class="px-3 py-4 text-sm text-gray-500 whitespace-nowrap">
                    <strong>{{ mail.subject }}</strong>
                  </td>
                  <td class="px-3 py-4 text-sm text-gray-500 whitespace-nowrap">{{ formatDate(mail.sentAt) }}</td>
                  <td>
                    <button class="px-2 border rounded cursor-pointer bg-gray-50 hover:bg-green-50"
                      @click="archiveMail(mail)">Archive</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <MailView v-if="modal" :mail="modal" @toggle="toggleModal" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>