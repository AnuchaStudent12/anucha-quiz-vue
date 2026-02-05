<template>
  <div class="container mt-4">
    <div class="row justify-content-center">
      <div class="col-12 col-md-8 col-lg-6">
        <div class="mb-3 d-flex align-items-center">
          <label class="me-2 fw-bold">ชื่อ-สกุล</label>
          <input type="text" class="form-control w-50" v-model="fullName" />
        </div>

        <!-- Quiz -->
        <div v-for="(q, qIndex) in quizData" :key="qIndex" class="mb-4">
          <div class="fw-bold mb-2">{{ qIndex + 1 }}. {{ q.question }}</div>

          <BFormRadioGroup
            v-model="userAnswers[qIndex]"
            :disabled="submitted"
            stacked
          >
            <BFormRadio
              v-for="(choice, cIndex) in q.choices"
              :key="cIndex"
              :value="cIndex + 1"
              class="mb-1 fs-6"
            >
              {{ choice }}
            </BFormRadio>
          </BFormRadioGroup>
        </div>

        <!-- Buttons -->
        <div class="row mt-4">
          <div class="col d-flex align-items-center gap-3">
            <BButton v-if="!submitted" variant="primary" @click="submitQuiz">
              ส่งคำตอบ
            </BButton>

            <BButton v-else variant="primary" @click="resetQuiz">
              สอบอีกครั้ง
            </BButton>

            <!-- Result -->
            <span v-if="submitted" class="fw-bold">
              คุณ {{ fullName }} ได้คะแนน ข้อที่ได้คะแนน : {{ score }}/{{
                quizData.length
              }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { BFormRadioGroup, BFormRadio, BButton } from "bootstrap-vue-next";

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

// ===== state =====
const quizData = ref([]);
const quizId = ref(null);

const fullName = ref("");
const userAnswers = ref([]);
const submitted = ref(false);
const score = ref(0);

// ===== load quiz =====
const loadQuiz = async () => {
  const res = await fetch(API_BASE_URL+"/api/Quiz");
  const data = await res.json();

  quizId.value = data.id ?? data[0].id;
  quizData.value = JSON.parse(data.quizData ?? data[0].quizData);

  userAnswers.value = Array(quizData.value.length).fill(null);
};

// ===== submit =====
const submitQuiz = async () => {
  const payload = {
    quizId: quizId.value,
    fullName: fullName.value,
    quizAnsData: userAnswers.value.join("|"),
  };

  const res = await fetch(API_BASE_URL + "/api/Quiz", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      accept: "text/plain",
    },
    body: JSON.stringify(payload),
  });

  const result = await res.json();

  score.value = result.score;
  fullName.value = result.fullName;
  submitted.value = true;
};

// ===== reset =====
const resetQuiz = () => {
  userAnswers.value = Array(quizData.value.length).fill(null);
  score.value = 0;
  submitted.value = false;
  fullName.value = "";
};

// ===== lifecycle =====
onMounted(loadQuiz);
</script>
