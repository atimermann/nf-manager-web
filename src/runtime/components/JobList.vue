<template>
  <DataTable
    :value="jobs"
    paginator
    :rows="20"
    sort-mode="multiple"
  >
    <Column sortable field="application" header="Projeto" />
    <Column sortable field="app" header="App" />
    <Column sortable field="controller" header="Controller" />
    <Column sortable field="name" header="Job" />
    <Column sortable field="scheduleText" header="Agendamento" />
    <Column sortable field="workers" header="Workers" />
    <Column sortable field="concurrency" header="Inst." />
    <Column sortable field="persistent" header="Pers." />
    <Column sortable field="status" header="Status" />
    <Column sortable field="errorCount" header="Erros">
      <template #body="{ data }">
        <div :style="data.errorCount > 0 ? 'background-color: var(--red-200)' : ''" class="text-center">
          {{ data.errorCount }}
        </div>
      </template>
    </Column>
    <Column>
      <template #body="{ data }">
        <!-- TODO: v-tooltip="'Overview'" ATIVAR NO PRIMEVUE  https://primevue.org/tooltip/   -->
        <Button
          type="button"
          icon="pi pi-book"
          text
          size="small"
          @click="toOverview(data.uuid)"
        />
      </template>
    </Column>
  </DataTable>
</template>

<script setup>

import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import Button from 'primevue/button'
import { io } from 'socket.io-client'
import cronstrue from 'cronstrue/i18n'

import { ref, useSocketTools, navigateTo } from '#imports'

const socket = useSocketTools.connect(io, '/jobs')

let jobsIndex = {}

socket.on('connect', () => {
  console.info('Connection with server ok')
})

socket.on('jobsList', jobInformation => {
  jobsIndex = jobInformation
  updateJobsList()
})

const jobs = ref(
  []
)

function updateJobsList () {
  jobs.value = []

  Object.keys(jobsIndex).forEach((key) => {
    const jobInfo = jobsIndex[key]

    let scheduleText = ''
    if (jobInfo.schedule && jobInfo.schedule !== 'now') {
      scheduleText = cronstrue.toString(jobInfo.schedule, { locale: 'pt_BR' })
    } else if (jobInfo.schedule === 'now') {
      scheduleText = 'Na inicialização'
    }

    jobs.value.push({
      uuid: jobInfo.uuid,
      name: jobInfo.name,
      application: jobInfo.applicationName,
      app: jobInfo.appName,
      controller: jobInfo.controllerName,
      scheduleText,
      workers: jobInfo.workers?.join(', '),
      concurrency: jobInfo.concurrency,
      persistent: jobInfo.persistent ? 'Sim' : 'Não',
      status: jobInfo.status,
      errorCount: jobInfo.errorCount
    })
  })
}

/**
 * Navega para página de detalhes do Job
 *
 * @param uuid    Identificador do Job
 *
 * @returns {Promise<void>}
 */
async function toOverview (uuid) {
  await navigateTo(`/job-overview/${uuid}`)
}

</script>

<style scoped>

</style>
