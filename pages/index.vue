<template>
  <div class="container mx-auto p-6 font-sans">
    <div class="text-center mb-8">
      <h1 class="text-3xl font-bold text-gray-800">Data Mahasiswa</h1>
    </div>

    <div class="flex justify-end mb-4">
      <button
        @click="openModal('add')"
        class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded shadow flex items-center gap-2 transition duration-200"
      >
        <span>+ Tambah Data</span>
      </button>
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
            <th class="py-3 px-6 text-center">Aksi</th>
          </tr>
        </thead>
        <tbody class="text-gray-600 text-sm font-light">
          <tr
            v-for="student in students"
            :key="student.id"
            class="border-b border-gray-200 hover:bg-gray-100 transition duration-150 ease-in-out"
          >
            <td class="py-3 px-6 text-left whitespace-nowrap font-medium">
              {{ student.id }}
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

            <td class="py-3 px-6 text-center">
              <div class="flex item-center justify-center space-x-2">
                <button
                  @click="openModal('edit', student)"
                  class="w-8 h-8 rounded-full bg-yellow-100 hover:bg-yellow-200 text-yellow-600 flex items-center justify-center transition"
                  title="Edit"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"
                    />
                  </svg>
                </button>
                <button
                  @click="deleteStudent(student.id)"
                  class="w-8 h-8 rounded-full bg-red-100 hover:bg-red-200 text-red-600 flex items-center justify-center transition"
                  title="Hapus"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                    />
                  </svg>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="mt-4 text-center text-xs text-gray-400">
      Total {{ students.length }} mahasiswa ditampilkan.
    </div>

    <div
      v-if="isModalOpen"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm"
    >
      <div
        class="bg-white rounded-lg shadow-xl w-full max-w-md p-6 transform transition-all"
      >
        <div class="flex justify-between items-center mb-4 border-b pb-2">
          <h3 class="text-xl font-bold text-gray-800">
            {{ formMode === "add" ? "Tambah Mahasiswa" : "Edit Mahasiswa" }}
          </h3>
          <button @click="closeModal" class="text-gray-500 hover:text-gray-700">
            ✕
          </button>
        </div>

        <form @submit.prevent="saveStudent">
          <div class="mb-4">
            <label class="block text-gray-700 text-sm font-bold mb-2"
              >Nama Lengkap</label
            >
            <input
              v-model="form.name"
              type="text"
              required
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Contoh: John Doe"
            />
          </div>

          <div class="mb-4">
            <label class="block text-gray-700 text-sm font-bold mb-2"
              >Email</label
            >
            <input
              v-model="form.email"
              type="email"
              required
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="email@example.com"
            />
          </div>

          <div class="mb-6">
            <label class="block text-gray-700 text-sm font-bold mb-2"
              >Course</label
            >
            <select
              v-model="form.course"
              required
              class="shadow border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option disabled value="">Pilih Course</option>
              <option value="JAVA">Java</option>
              <option value="JAVASCRIPT">JavaScript</option>
              <option value="PYTHON">Python</option>
              <option value="GOLANG">Golang</option>
              <option value="C++">C++</option>
            </select>
          </div>

          <div class="flex justify-end gap-2">
            <button
              type="button"
              @click="closeModal"
              class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded focus:outline-none"
            >
              Batal
            </button>
            <button
              type="submit"
              class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none shadow-lg"
            >
              {{ isSaving ? "Menyimpan..." : "Simpan" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "StudentPage",

  data() {
    return {
      students: [],
      // State untuk Modal
      isModalOpen: false,
      formMode: "add", // 'add' atau 'edit'
      isSaving: false,
      editId: null,

      // Data Form
      form: {
        name: "",
        email: "",
        course: "",
      },
    };
  },

  async fetch() {
    try {
      this.students = await this.$axios.$get("/api/students");
    } catch (err) {
      console.error(err);
      // Opsi: Anda bisa throw error, tapi lebih baik handle di UI saja agar app tidak crash
      // throw new Error("Gagal load data");
    }
  },

  methods: {
    // 1. Membuka Modal
    openModal(mode, student = null) {
      this.formMode = mode;
      this.isModalOpen = true;

      if (mode === "edit" && student) {
        this.editId = student.id;
        // Copy data agar tidak merubah tampilan tabel secara realtime sebelum save
        this.form = { ...student };
      } else {
        // Reset form untuk mode add
        this.editId = null;
        this.form = {
          name: "",
          email: "",
          course: "",
        };
      }
    },

    // 2. Menutup Modal
    closeModal() {
      this.isModalOpen = false;
    },

    // 3. Simpan Data (Create / Update)
    async saveStudent() {
      this.isSaving = true;
      try {
        if (this.formMode === "add") {
          // --- CREATE (POST) ---
          await this.$axios.$post("/api/students", this.form);
          alert("Berhasil menambah data!");
        } else {
          // --- UPDATE (PUT/PATCH) ---
          await this.$axios.$put(`/api/students/${this.editId}`, this.form);
          alert("Berhasil update data!");
        }

        // Refresh data tabel dan tutup modal
        this.$fetch();
        this.closeModal();
      } catch (err) {
        console.error(err);
        alert("Gagal menyimpan data. Cek console.");
      } finally {
        this.isSaving = false;
      }
    },

    // 4. Hapus Data (Delete)
    async deleteStudent(id) {
      if (!confirm("Apakah Anda yakin ingin menghapus data ini?")) return;

      try {
        // --- DELETE ---
        await this.$axios.$delete(`/api/students/${id}`);
        // Refresh data tabel
        this.$fetch();
        alert("Data berhasil dihapus");
      } catch (err) {
        console.error(err);
        alert("Gagal menghapus data.");
      }
    },

    // Helper untuk warna badge (Sama seperti kode asli)
    getBadgeColor(course) {
      if (!course) return "bg-gray-400";
      const c = course.toLowerCase();
      if (c.includes("go")) return "bg-blue-500";
      if (c.includes("python")) return "bg-yellow-500 text-black";
      if (c.includes("java") && !c.includes("script")) return "bg-red-500";
      if (c.includes("script")) return "bg-green-500";
      if (c.includes("c++")) return "bg-purple-600";
      return "bg-gray-500";
    },
  },
};
</script>
