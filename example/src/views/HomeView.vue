<script lang="ts" setup>
import { inject, ref, onMounted } from "vue"
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome"

import { library } from "@fortawesome/fontawesome-svg-core"
import { fas } from "@fortawesome/free-solid-svg-icons"
import { time_ago } from "../utils"

import MainLayout from "../layouts/MainLayout.vue"
import {
  plan1_source,
  plan1_source_json,
  plan1_query,
  plan2_source,
  plan2_query,
  plan3_source,
  plan3_query,
  plan4_source,
  plan5_source,
  plan5_query,
  plan6_source,
  plan7_source,
  plan8_source,
  plan_parallel_source,
  plan_parallel_2_source,
  plan_parallel_2_query,
  plan_trigger_source,
  plan_trigger_query,
  plan_trigger_2_source,
  plan_trigger_2_query,
} from "../samples.ts"

const setPlanData = inject("setPlanData")
library.add(fas)
interface Storage {
  loadSavedPlans(): Plan[]
  storeSavedPlans(plans: Plan[]): void
}

class LocalPlanStorage implements Storage {
  private static readonly KEY = "plans"
  loadSavedPlans(): Plan[] {
    return JSON.parse(localStorage.getItem(LocalPlanStorage.KEY) || "[]")
  }

  storeSavedPlans(plans: Plan[]): void {
    localStorage.setItem(LocalPlanStorage.KEY, JSON.stringify(plans))
  }
}

const planInput = ref<string>("")
const queryInput = ref<string>("")
const queryName = ref<string>("")
const draggingPlan = ref<boolean>(false)
const draggingQuery = ref<boolean>(false)
const planStorage: Storage = new LocalPlanStorage()

interface Plan extends Array<string | undefined> {
  0: string /** name of the plan */
  1: string /** Text or JSON representation of the Plan */
  2: string /** SQL Query associated with the Plan */
  3?: string /** created at - time as ISO-8601 String */
}

const samples = ref<Plan[]>([
  ["Example 1 TEXT", plan1_source, plan1_query],
  ["Example 1 JSON", plan1_source_json, plan1_query],
  ["Example 2", plan2_source, plan2_query],
  ["Example 3", plan3_source, plan3_query],
  ["Example 4", plan4_source, ""],
  ["Example 5", plan5_source, plan5_query],
  ["With subplan", plan6_source, ""],
  ["With CTE", plan7_source, ""],
  ["Very large plan", plan8_source, ""],
  ["With trigger", plan_trigger_2_source, plan_trigger_2_query],
  ["With trigger (plain text)", plan_trigger_source, plan_trigger_query],
  ["Parallel (verbose)", plan_parallel_source, ""],
  ["Parallel (4 workers)", plan_parallel_2_source, plan_parallel_2_query],
])

let savedPlans = ref<Plan[]>()
let currentPlanIndex = ref<number>(-1)

function submitPlan() {
  let newPlan: Plan = ["", "", ""]
  newPlan[0] =
    queryName.value ||
    "New Plan - " +
      new Date().toLocaleString("en-IN", {
        dateStyle: "medium",
        timeStyle: "medium",
      })
  newPlan[1] = planInput.value
  newPlan[2] = queryInput.value
  newPlan[3] = new Date().toISOString()
  savePlanData(newPlan)

  setPlanData(...newPlan)
}

function savePlanData(sample: Plan) {
  savedPlans.value = planStorage.loadSavedPlans()
  if (currentPlanIndex.value >= 0) {
    savedPlans.value[currentPlanIndex.value] = sample
  } else {
    savedPlans.value.unshift(sample)
  }
  planStorage.storeSavedPlans(savedPlans.value)
}

onMounted(() => {
  const textAreas = document.getElementsByTagName("textarea")
  Array.prototype.forEach.call(textAreas, (elem: HTMLInputElement) => {
    elem.placeholder = elem.placeholder.replace(/\\n/g, "\n")
  })
  const noHashURL = window.location.href.replace(/#.*$/, "")
  window.history.replaceState("", document.title, noHashURL)
  savedPlans.value = planStorage.loadSavedPlans()
})

/**
 * Loads the plan details in the Form.
 * Can be used for editing the details.
 *
 * @param plan
 * @param index
 */
function loadPlan(plan?: Plan, index?: number) {
  if (!plan) {
    return
  }

  if (index != null && index != undefined) {
    currentPlanIndex.value = index
  } else {
    currentPlanIndex.value = -1
  }

  queryName.value = plan[0]
  planInput.value = plan[1]
  queryInput.value = plan[2]
}

/**
 * Opens the Plan detail view for the given Plan
 * @param plan
 */
function openPlan(plan: Plan) {
  setPlanData(plan[0], plan[1], plan[2])
}

function editPlan(index: number) {
  loadPlan(savedPlans?.value && savedPlans?.value[index], index)
}

function deletePlan(index: number) {
  savedPlans.value?.splice(index, 1)
  planStorage.storeSavedPlans(savedPlans.value || [])
}

function handleDrop(event: DragEvent) {
  const input = event.srcElement
  if (!(input instanceof HTMLTextAreaElement)) {
    return
  }
  draggingPlan.value = false
  draggingQuery.value = false
  if (!event.dataTransfer) {
    return
  }
  const file = event.dataTransfer.files[0]
  const reader = new FileReader()
  reader.onload = () => {
    if (reader.result instanceof ArrayBuffer) {
      return
    }
    input.value = reader.result || ""
    input.dispatchEvent(new Event("input"))
  }
  reader.readAsText(file)
}
</script>

<template>
  <main-layout>
    <div class="container">
      <div class="alert alert-warning">
        This is a tool to visualize Postgres EXPLAIN plans. It is adapted from
        <a href="https://github.com/dalibo/pev2">PEV2</a>. It is serverless and
        stores your plans in your browser local storage only.
      </div>
      <div class="row form-group">
        <div class="col d-flex">
          <div class="text-muted">
            For best results, use
            <code>
              EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON)
            </code>
            <br />
            <em>psql</em> users can export the plan to a file using
            <code>psql -XqAt -f explain.sql > analyze.json</code>
          </div>
          <div class="dropdown ml-auto">
            <button
              class="btn btn-secondary dropdown-toggle"
              type="button"
              id="dropdownMenuButton"
              data-toggle="dropdown"
              aria-haspopup="true"
              aria-expanded="false"
            >
              Sample Plans
            </button>
            <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
              <a
                v-for="(sample, index) in samples"
                :key="index"
                class="dropdown-item"
                v-on:click.prevent="loadPlan(sample)"
                href
              >
                {{ sample[0] }}
              </a>
            </div>
          </div>
        </div>
      </div>
      <div class="row form-group">
        <div class="col-sm-7">
          <form v-on:submit.prevent="submitPlan">
            <div class="form-group">
              <label for="planInput">
                Plan <span class="small text-muted">(text or JSON)</span>
              </label>
              <textarea
                :class="['form-control', draggingPlan ? 'dropzone-over' : '']"
                id="planInput"
                rows="8"
                v-model="planInput"
                @dragenter="draggingPlan = true"
                @dragleave="draggingPlan = false"
                @drop.prevent="handleDrop"
                placeholder="Paste execution plan\nOr drop a file"
              >
              </textarea>
            </div>
            <div class="form-group">
              <label for="queryInput">
                Query <span class="small text-muted">(optional)</span>
              </label>
              <textarea
                :class="['form-control', draggingQuery ? 'dropzone-over' : '']"
                id="queryInput"
                rows="4"
                v-model="queryInput"
                @dragenter="draggingQuery = true"
                @dragleave="draggingQuery = false"
                @drop.prevent="handleDrop"
                placeholder="Paste corresponding SQL query\nOr drop a file"
              >
              </textarea>
            </div>
            <div class="form-group">
              <label for="queryName">
                Query Name <span class="small text-muted">(optional)</span>
              </label>
              <input
                type="text"
                :class="['form-control']"
                id="queryName"
                v-model="queryName"
                placeholder="Give a name for this Query"
              />
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
        </div>

        <div class="col-sm-5 mb-4 mt-4 mt-md-0">
          <label> Saved Plans </label>
          <ul class="list-group" v-cloak>
            <li
              class="list-group-item px-2 py-1"
              v-for="(plan, index) in savedPlans"
              :key="index"
            >
              <div class="row">
                <div class="col">
                  <button
                    class="btn btn-sm btn-outline-secondary py-0 m-2 float-right"
                    title="Remove plan from list"
                    v-on:click.prevent="deletePlan(index)"
                  >
                    <font-awesome-icon icon="trash"></font-awesome-icon>
                  </button>
                  <button
                    class="btn btn-sm btn-outline-secondary py-0 m-2 float-right"
                    title="Edit plan details"
                    v-on:click.prevent="editPlan(index)"
                  >
                    <font-awesome-icon icon="edit"></font-awesome-icon>
                  </button>
                  <a
                    v-on:click.prevent="openPlan(plan)"
                    href=""
                    title="Open the plan details"
                  >
                    {{ plan[0] }}
                  </a>
                </div>
              </div>

              <div class="row">
                <div class="col-6">
                  <small class="text-muted">
                    created
                    <span :title="plan[3]?.toString()">
                      {{ time_ago(plan[3]) }}
                    </span>
                  </small>
                </div>
                <div class="col-6 text-right"></div>
              </div>
            </li>
          </ul>
          <p class="text-muted text-center" v-if="!savedPlans?.length" v-cloak>
            <em> You haven't saved any plan yet. </em>
          </p>
        </div>
      </div>
    </div>
  </main-layout>
</template>
