<template>
  <div class="flavorio-container">
    <h1>Flavorio</h1>
    <div style="margin-bottom: 1em;">
      <input type="file" accept=".xml" @change="onFileChange" />
    </div>
    <div style="margin-bottom: 1em;">
      <label>
        <input type="radio" value="teacher" v-model="viewMode" :disabled="!xmlData" /> Por Docente
      </label>
      <label style="margin-left: 1em;">
        <input type="radio" value="classroom" v-model="viewMode" :disabled="!xmlData" /> Por Aula
      </label>
    </div>
    <div style="margin-bottom: 1em;">
      <button v-if="(selectedTeacher || selectedClassroom)" @click="exportPDF">Exportar horario a PDF</button>
    </div>
    <table v-if="viewMode === 'teacher' && rows.length">
      <thead>
        <tr>
          <th>Nombre</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, idx) in rows" :key="idx">
          <td>
            <a href="#" @click.prevent="selectTeacher(row)">{{ row.name }}</a>
          </td>
        </tr>
      </tbody>
    </table>
    <table v-else-if="viewMode === 'classroom' && classroomRows.length">
      <thead>
        <tr>
          <th>Nombre</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, idx) in classroomRows" :key="idx">
          <td>
            <a href="#" @click.prevent="selectClassroom(row)">{{ row.name }}</a>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-else-if="xmlData">No hay datos para mostrar.</div>
    <div v-else>Por favor, carga un archivo XML.</div>

    <div v-if="selectedTeacher">
      <h2>Horario de {{ selectedTeacher.name }}</h2>
      <div class="horario-scroll">
        <table class="horario-table">
          <thead>
            <tr>
              <th>Hora</th>
              <th v-for="d in weekDays" :key="d.value">{{ d.label }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="hour in hours" :key="hour">
              <td>{{ hour }}:00</td>
              <td v-for="d in weekDays" :key="d.value">
                <div v-if="horarioGrid[hour] && horarioGrid[hour][d.value]">
                  <div><strong>{{ horarioGrid[hour][d.value].subjectName }}</strong></div>
                  <div>{{ horarioGrid[hour][d.value].className }}</div>
                  <div>{{ horarioGrid[hour][d.value].classroomName }}</div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div v-if="selectedClassroom">
      <h2>Horario del Aula {{ selectedClassroom.name }}</h2>
      <div class="horario-scroll">
        <table class="horario-table">
          <thead>
            <tr>
              <th>Hora</th>
              <th v-for="d in weekDays" :key="d.value">{{ d.label }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="hour in hours" :key="hour">
              <td>{{ hour }}:00</td>
              <td v-for="d in weekDays" :key="d.value">
                <div v-if="horarioGrid[hour] && horarioGrid[hour][d.value]">
                  <div><strong>{{ horarioGrid[hour][d.value].subjectName }}</strong></div>
                  <div>{{ horarioGrid[hour][d.value].className }}</div>
                  <div>{{ horarioGrid[hour][d.value].teacherName }}</div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import jsPDF from 'jspdf';
import 'jspdf-autotable';
const weekDays = [
  { value: '0', label: 'Lunes' },
  { value: '1', label: 'Martes' },
  { value: '2', label: 'Miércoles' },
  { value: '3', label: 'Jueves' },
  { value: '4', label: 'Viernes' },
  { value: '5', label: 'Sábado' },
  { value: '6', label: 'Domingo' }
];
const hours = Array.from({ length: 15 }, (_, i) => i + 7); // 7 a 21

export default {
  name: 'Flavorio',
  data() {
    return {
      viewMode: 'teacher',
      headers: ['id', 'name', 'short'],
      rows: [],
      selectedTeacher: null,
      selectedClassroom: null,
      schedule: [],
      horarioGrid: {},
      xmlData: null,
      classroomHeaders: ['id', 'name', 'short'],
      classroomRows: [],
      weekDays,
      hours
    };
  },
  methods: {
    exportPDF() {
      const doc = new jsPDF({ orientation: 'landscape', unit: 'mm', format: 'a4' });
      let title = '';
      let tableBody = [];
      let columns = [
        { header: 'Hora', dataKey: 'hora' },
        ...this.weekDays.map(d => ({ header: d.label, dataKey: d.value }))
      ];
      if (this.selectedTeacher) {
        title = `Horario de ${this.selectedTeacher.name}`;
        for (let hour of this.hours) {
          let row = { hora: `${hour}:00` };
          for (let d of this.weekDays) {
            if (this.horarioGrid[hour] && this.horarioGrid[hour][d.value]) {
              const cell = this.horarioGrid[hour][d.value];
              row[d.value] = `${cell.subjectName}\n${cell.className}\n${cell.classroomName}`;
            } else {
              row[d.value] = '';
            }
          }
          tableBody.push(row);
        }
      } else if (this.selectedClassroom) {
        title = `Horario del Aula ${this.selectedClassroom.name}`;
        for (let hour of this.hours) {
          let row = { hora: `${hour}:00` };
          for (let d of this.weekDays) {
            if (this.horarioGrid[hour] && this.horarioGrid[hour][d.value]) {
              const cell = this.horarioGrid[hour][d.value];
              row[d.value] = `${cell.subjectName}\n${cell.className}\n${cell.teacherName}`;
            } else {
              row[d.value] = '';
            }
          }
          tableBody.push(row);
        }
      }
      doc.setFontSize(14);
      doc.text(title, 14, 14);
      doc.autoTable({
        startY: 20,
        head: [columns.map(col => col.header)],
        body: tableBody.map(row => columns.map(col => row[col.dataKey] || '')),
        styles: {
          fontSize: 6,
          cellPadding: 1.2,
          overflow: 'linebreak',
          valign: 'middle',
        },
        headStyles: { fillColor: [40, 40, 40], textColor: 255, fontSize: 7, cellPadding: 1 },
        bodyStyles: { valign: 'middle', fontSize: 6, cellPadding: 1.2, overflow: 'linebreak' },
        columnStyles: {
          // Hora columna un poco más ancha
          0: { cellWidth: 15 },
          // Todas las columnas de días con ancho fijo pequeño
          1: { cellWidth: 28 },
          2: { cellWidth: 28 },
          3: { cellWidth: 28 },
          4: { cellWidth: 28 },
          5: { cellWidth: 28 },
          6: { cellWidth: 28 },
          7: { cellWidth: 28 },
        },
        theme: 'grid',
        tableWidth: 'auto',
        pageBreak: 'avoid',
        didDrawPage: function (data) {
          // No hacer nada, solo evitar saltos de página
        }
      });
      doc.save(`${title}.pdf`);
    },
    onFileChange(e) {
      const file = e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (event) => {
        const xmlStr = event.target.result;
        const parser = new window.DOMParser();
        const xml = parser.parseFromString(xmlStr, 'text/xml');
        this.xmlData = xml;
        // Docentes
        const teacherNodes = Array.from(xml.getElementsByTagName('teacher'));
        this.rows = teacherNodes.map(node => ({
          id: node.getAttribute('id'),
          name: node.getAttribute('name'),
          short: node.getAttribute('short')
        }));
        // Aulas
        const classroomNodes = Array.from(xml.getElementsByTagName('classroom'));
        this.classroomRows = classroomNodes.map(node => ({
          id: node.getAttribute('id'),
          name: node.getAttribute('name'),
          short: node.getAttribute('short')
        }));
        // Limpiar selección previa
        this.selectedTeacher = null;
        this.selectedClassroom = null;
        this.schedule = [];
        this.horarioGrid = {};
      };
      reader.readAsText(file);
    },
    selectTeacher(teacher) {
      this.selectedTeacher = teacher;
      this.selectedClassroom = null;
      if (!this.xmlData) return;
      // Obtener cards del profesor
      const cards = Array.from(this.xmlData.getElementsByTagName('card'));
      // Filtrar por teacherids
      const teacherId = teacher.id;
      const teacherCards = cards.filter(card => {
        const ids = card.getAttribute('teacherids') || '';
        return ids.split(',').includes(teacherId);
      });
      // Diccionarios para nombres
      const classMap = {};
      Array.from(this.xmlData.getElementsByTagName('class')).forEach(c => {
        classMap[c.getAttribute('id')] = c.getAttribute('name');
      });
      const subjectMap = {};
      Array.from(this.xmlData.getElementsByTagName('subject')).forEach(s => {
        subjectMap[s.getAttribute('id')] = s.getAttribute('name');
      });
      const classroomMap = {};
      Array.from(this.xmlData.getElementsByTagName('classroom')).forEach(r => {
        classroomMap[r.getAttribute('id')] = r.getAttribute('name');
      });
      // Construir grid horario
      const grid = {};
      for (let h = 7; h <= 21; h++) grid[h] = {};
      teacherCards.forEach(card => {
        const day = card.getAttribute('day');
        const period = parseInt(card.getAttribute('period'));
        if (period < 7 || period > 21) return;
        grid[period][day] = {
          subjectName: subjectMap[card.getAttribute('subjectid')] || card.getAttribute('subjectid'),
          className: (card.getAttribute('classids') || '').split(',').map(id => classMap[id] || id).join(', '),
          classroomName: classroomMap[card.getAttribute('classroomids')] || card.getAttribute('classroomids')
        };
      });
      this.horarioGrid = grid;
    },
    selectClassroom(classroom) {
      this.selectedClassroom = classroom;
      this.selectedTeacher = null;
      if (!this.xmlData) return;
      // Obtener cards del aula
      const cards = Array.from(this.xmlData.getElementsByTagName('card'));
      // Filtrar por classroomids
      const classroomId = classroom.id;
      const classroomCards = cards.filter(card => {
        const ids = card.getAttribute('classroomids') || '';
        return ids.split(',').includes(classroomId);
      });
      // Diccionarios para nombres
      const classMap = {};
      Array.from(this.xmlData.getElementsByTagName('class')).forEach(c => {
        classMap[c.getAttribute('id')] = c.getAttribute('name');
      });
      const subjectMap = {};
      Array.from(this.xmlData.getElementsByTagName('subject')).forEach(s => {
        subjectMap[s.getAttribute('id')] = s.getAttribute('name');
      });
      const teacherMap = {};
      Array.from(this.xmlData.getElementsByTagName('teacher')).forEach(t => {
        teacherMap[t.getAttribute('id')] = t.getAttribute('name');
      });
      // Construir grid horario
      const grid = {};
      for (let h = 7; h <= 21; h++) grid[h] = {};
      classroomCards.forEach(card => {
        const day = card.getAttribute('day');
        const period = parseInt(card.getAttribute('period'));
        if (period < 7 || period > 21) return;
        grid[period][day] = {
          subjectName: subjectMap[card.getAttribute('subjectid')] || card.getAttribute('subjectid'),
          className: (card.getAttribute('classids') || '').split(',').map(id => classMap[id] || id).join(', '),
          teacherName: (card.getAttribute('teacherids') || '').split(',').map(id => teacherMap[id] || id).join(', ')
        };
      });
      this.horarioGrid = grid;
    }
  }
};
</script>

<style scoped>
.flavorio-container {
  padding: 32px 24px 24px 24px;
  box-sizing: border-box;
}
.horario-scroll {
  overflow-x: auto;
  overflow-y: auto;
  margin-bottom: 1em;
}
.horario-table {
  border-collapse: collapse;
  width: 100%;
  min-width: 900px;
  margin-top: 1em;
}
.horario-table th, .horario-table td {
  border: 1px solid #ccc;
  padding: 6px 8px;
  text-align: center;
  min-width: 90px;
  font-size: 13px;
}
.horario-table th {
  background: #f0f0f0;
}
</style> 