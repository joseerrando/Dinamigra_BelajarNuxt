<template>
  <div class="container mx-auto p-6 font-sans">
    <div class="text-center mb-8">
      <h1 class="text-3xl font-bold text-gray-800">Data Mahasiswa</h1>
    </div>

    <div
      class="bg-green-100 border border-green-400 text-white px-4 py-3 rounded mb-6"
    >
      <button>Test Button</button>
    </div>
    <div
      v-if="$fetchState.pending"
      class="text-center text-blue-600 animate-pulse text-xl font-semibold"
    >
      Sedang memuat data...
    </div>

    <div
      v-else-if="$fetchState.error"
      class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative text-center"
    >
      <strong class="font-bold">Error!</strong>
      <span class="block sm:inline">
        Gagal mengambil data API. Pastikan server backend nyala.</span
      >
    </div>

    <div v-else class="overflow-x-auto shadow-lg rounded-lg">
      <table class="min-w-full bg-white">
        <thead>
          <tr class="bg-gray-800 text-white uppercase text-sm leading-normal">
            <th class="py-3 px-6 text-left">ID</th>
            <th class="py-3 px-6 text-left">Nama Lengkap</th>
            <th class="py-3 px-6 text-left">Email</th>
            <th class="py-3 px-6 text-left">Course</th>
          </tr>
        </thead>
        <tbody class="text-gray-600 text-sm font-light">
          <tr
            v-for="student in students"
            :key="student.ID"
            class="border-b border-gray-200 hover:bg-gray-100 transition duration-150 ease-in-out"
          >
            <td class="py-3 px-6 text-left whitespace-nowrap font-medium">
              {{ student.ID }}
            </td>

            <td class="py-3 px-6 text-left">
              <div class="flex items-center">
                <span class="font-bold text-gray-700">
                  {{ student.name ? student.name : "(Tanpa Nama)" }}
                </span>
              </div>
            </td>

            <td class="py-3 px-6 text-left">
              {{ student.email ? student.email : "-" }}
            </td>

            <td class="py-3 px-6 text-left">
              <span
                :class="[
                  'py-1 px-3 rounded-full text-xs text-white font-bold uppercase shadow-sm',
                  getBadgeColor(student.course),
                ]"
              >
                {{ student.course ? student.course : "N/A" }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="mt-4 text-center text-xs text-gray-400">
      Total {{ students.length }} mahasiswa ditampilkan.
    </div>
  </div>
</template>

<script>
export default {
  name: "StudentPage",

  data() {
    return {
      students: [],
    };
  },

  async fetch() {
    try {
      this.students = await this.$axios.$get("/api/students");
    } catch (err) {
      console.error(err);
      throw new Error("Gagal load data");
    }
  },

  methods: {
    // Return class warna Tailwind
    getBadgeColor(course) {
      if (!course) return "bg-gray-400";

      const c = course.toLowerCase();
      // Menggunakan palette warna Tailwind (500 biasanya standar cerah)
      if (c.includes("go")) return "bg-blue-500";
      if (c.includes("python")) return "bg-yellow-500 text-black"; // Text hitam biar kontras
      if (c.includes("java") && !c.includes("script")) return "bg-red-500";
      if (c.includes("script")) return "bg-green-500";
      if (c.includes("c++")) return "bg-purple-600";

      return "bg-gray-500"; // Default
    },
  },
};
</script>
