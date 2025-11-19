<script setup>
import { defineProps, defineEmits, ref, nextTick } from 'vue'

const props = defineProps({
  name: { type: String, default: '' },
  active: { type: Boolean, default: false },
  editable: { type: Boolean, default: true },
  readonly: { type: Boolean, default: false },
  side: { type: String, default: 'right' }, // 'left' or 'right'
	placeholder: { type: String, default: '' }, // Letter placeholder (A-P)
	loser: { type: Boolean, default: false }
})
const emit = defineEmits(['select', 'rename'])

const isEditing = ref(false)
const localName = ref(props.name)
const inputRef = ref(null)

// click on check button -> emit 'select' to advance winner
function onSelectWinner(evt) {
  evt.stopPropagation() // prevent triggering startEdit
  emit('select')
}

// click on main area -> start editing
function startEdit() {
  if (!props.editable || props.readonly) return
  localName.value = props.name || ''
  isEditing.value = true
  nextTick(() => {
    if (inputRef.value) inputRef.value.focus()
  })
}

function saveEdit() {
  const newName = localName.value == null ? '' : String(localName.value).trim()
  emit('rename', newName === '' ? null : newName)
  isEditing.value = false
}

function cancelEdit() {
  isEditing.value = false
  localName.value = props.name || ''
}

function onKeydown(evt) {
  if (evt.key === 'Enter') {
    saveEdit()
  } else if (evt.key === 'Escape') {
    cancelEdit()
  }
}
</script>

<template>
	<div class="team-btn-wrapper">
		<div
			v-if="!isEditing"
			class="team-btn"
			:class="{ active: active, readonly: readonly, 'left-side': side === 'left', loser: loser }"
		>
			<!-- Small trophy button: advance winner (left side) -->
			<button v-if="side === 'left'" class="check-btn check-btn-left" @click="onSelectWinner" title="Avanzar ganador" aria-label="Marcar como ganador">
				🏆
			</button>
			<!-- Main clickable area: edit name -->
			<div class="name-area" @click="startEdit" :title="(!readonly && editable) ? 'Click para editar' : ''">
				<span class="label" v-if="name">{{ name }}</span>
				<span class="placeholder" v-else>{{ placeholder }}</span>
			</div>
			<!-- Small trophy button: advance winner (right side) -->
			<button v-if="side === 'right'" class="check-btn check-btn-right" @click="onSelectWinner" title="Avanzar ganador" aria-label="Marcar como ganador">
				🏆
			</button>
		</div>

		<input
			v-else
			class="team-input"
			v-model="localName"
			@blur="saveEdit"
			@keydown="onKeydown"
			ref="inputRef"
			placeholder="Nombre del participante"
			aria-label="Editar nombre del participante"
		/>
	</div>
</template>

<style scoped>
.team-btn-wrapper{
	position: relative;
}

.team-btn{
		/* rectangular: wider than tall */
		width: clamp(130px, 15vw, 220px);
		height: clamp(50px, 5.5vw, 90px);
		display:flex;
		align-items:stretch;
		justify-content:space-between;
		/* Arcade-style thin neon border using background-clip trick */
		background:
			linear-gradient(#ffffff, #ffffff) padding-box,
			linear-gradient(135deg, #10b981 0%, #3b82f6 50%, #f59e0b 100%) border-box;
		border: 2px solid transparent; /* keep it thin */
		padding: 0;
		border-radius: 10px;
		text-align: center;
		color: #042022;
		font-weight:700;
		transition: transform .15s ease, box-shadow .2s ease, filter .2s ease;
		overflow: hidden;
		/* subtle outer glow for arcade look (not too thick) */
		box-shadow:
			0 2px 10px rgba(16,185,129,0.18),
			0 0 14px rgba(16,185,129,0.18);
		box-sizing: border-box;
		position: relative;
}

.name-area{
	flex:1;
	display:flex;
	align-items:center;
	justify-content:center;
	cursor:text; /* text cursor to indicate editable */
	padding:6px 8px;
	transition: background 0.2s ease;
	overflow: hidden; /* hide overflow but allow wrapping inside label */
}

.name-area:hover{
	background: rgba(16,185,129,0.05);
}

.team-btn.readonly .name-area {
	cursor: default; /* no text cursor for readonly */
}

.team-btn.readonly .name-area:hover {
	background: transparent; /* no hover effect for readonly */
}

.name-area .label{ 
	display:block; 
	width:100%; 
	font-size: clamp(16px, 1.9vw, 28px); 
	line-height:1.3; 
	/* allow line breaks instead of ellipsis */
	white-space: normal;
	word-wrap: break-word;
	overflow: hidden;
	text-align: center;
}

.name-area .placeholder{
	display:block; 
	width:100%; 
	font-size: clamp(22px, 2.6vw, 36px); 
	line-height:1.3; 
	text-align: center;
	opacity: 0.65;
	color: #000000;
	font-weight: 900;
	font-family: 'Arial Black', 'Impact', sans-serif;
	text-transform: uppercase;
	letter-spacing: 2px;
	text-shadow: 2px 2px 0px rgba(16, 185, 129, 0.3);
}

/* Small trophy button */
.check-btn{
	width: clamp(36px, 3.8vw, 52px);
	height: 100%;
	display:flex;
	align-items:center;
	justify-content:center;
	background: rgba(16,185,129,0.12);
	border:none;
	font-size: clamp(18px, 2vw, 28px);
	cursor:pointer;
	transition: all 0.2s ease;
}

/* Trophy button on the right edge */
.check-btn-right{
	border-left: 1px solid rgba(16,185,129,0.2);
}

/* Trophy button on the left edge */
.check-btn-left{
	border-right: 1px solid rgba(16,185,129,0.2);
}

.check-btn:hover{
	background: rgba(16,185,129,0.25);
	transform: scale(1.1);
}

.team-btn:hover{
		transform: translateY(-2px);
		box-shadow:
			0 6px 16px rgba(16,185,129,0.28),
			0 0 20px rgba(16,185,129,0.26);
}

.team-btn.active{ 
		/* Keep the neon frame but soften fill */
		background:
			linear-gradient(#ecfdf5, #ecfdf5) padding-box,
			linear-gradient(135deg, #10b981 0%, #3b82f6 50%, #f59e0b 100%) border-box;
		color:#064e3b; 
		box-shadow:
			0 8px 22px rgba(16,185,129,0.30),
			0 0 26px rgba(16,185,129,0.28);
}

/* Loser visual state */
.team-btn.loser{
	/* Override arcade frame for losers */
	background: #e5e7eb; /* Tailwind gray-200 */
	color: #6b7280; /* gray-500 */
	filter: grayscale(1);
	opacity: 0.9;
	border: 2px solid rgba(107,114,128,0.35);
	box-shadow: none;
	pointer-events: none; /* prevent further interaction */
}

.team-btn.loser .label{
	text-decoration: line-through;
	text-decoration-thickness: 3px;
}

.team-input{
	width: clamp(130px, 15vw, 220px);
	height: clamp(50px, 5.5vw, 90px);
	padding: 8px 10px;
	border-radius: 8px;
	border: 2px solid rgba(16,185,129,0.18);
	font-weight:700;
	font-size: clamp(12px, 1.2vw, 16px);
	outline:none;
	box-sizing: border-box;
}
</style>
*** End Patch
