<script setup>
import { Input } from 'frappe-ui';
import { Button } from 'frappe-ui';
import { computed, reactive, ref } from 'vue';
import { createDocumentResource,createListResource } from 'frappe-ui';
const props = defineProps({
	id:{
		type:String,
	}
})

const currentStep = ref(0)

const movie = ref(props.id)

const movieData = createDocumentResource({
	doctype: "Movie",
	name: movie.value,
	fields: "[*]",
	auto: true
})

const movieDoc = computed(() => movieData.doc)

const theatres = createListResource({
	doctype: "Theater Show",
	fields: ["theatre","start_time","name"],
	auto: true,
	transform: (shows) =>{
		if (!shows.length) return

		const groupedShows = {}
		for(let show of shows) {
			if (!groupedShows[show.theatre]) {
				groupedShows[show.theatre] = []
			}
			groupedShows[show.theatre].push(show)
		}
		return groupedShows
	},
})

function getSeatStructure(alphabets, numbers) {
	const structure = {}
	for (const alphabet of alphabets) {
		structure[alphabet] = []
		for (const number of numbers) {
			structure[alphabet].push([number,"Available"])
		}
	}
	return structure
}
const seatStructure = reactive(getSeatStructure(["A", "B", "C", "D", "E"], [1, 2, 3, 4, 5, 6, 7]))

function selectSeat(row, seat) {
	// make the seat booked in structure
	if (hasSelectedCorrectNumberOfSeats.value) {
		return
	}
	bookingData.selectedSeats.push(`${row}${seat}`)
	seatStructure[row][seat - 1][1] = "Selected"

}

const hasSelectedCorrectNumberOfSeats = computed(() => {
	return bookingData.selectedSeats.length === bookingData.numberOfSeats
})


const bookingData = reactive({
	numberOfSeats:1,
	date: new Date().toISOString().slice(0,10),
	seats:seatStructure,
	selectedSeats: [],
	show:null
})


const setNumberOfSeats = (index) =>{
	bookingData.numberOfSeats = index
}

const nextButtonEnabled = computed(() => {
	if (currentStep.value === 1) {
		return bookingData.numberOfSeats
	}
	if (currentStep.value === 2) {
		return bookingData.date
	}
	if (currentStep.value === 3) {
		return bookingData.show
	}
	if (currentStep.value === 4) {
		return hasSelectedCorrectNumberOfSeats.value
	}
})

const movieBooking = createListResource({
	doctype: "Ticket Booking",
	insert: {
		onSuccess: () => {
			currentStep.value++
		}
	},
})

const moveToNextStep = () => {
	if (currentStep.value === 4) {
		// make the booking and then move to next step
		movieBooking.insert.submit({
			movie: movie.value,
			date: bookingData.date,
			show: bookingData.show,
			number_of_tickets: bookingData.numberOfSeats,
			seats_booked: JSON.stringify(bookingData.selectedSeats),
		})
		return
	}
	currentStep.value++
}



</script>

<template>
	<div v-if="!movieData.loading && movieDoc" class="p-8">
		<h1 class=" text-gray-900 font-bold text-[32px] ">{{movieDoc.title}}</h1>

		<div class=" mt-11 flex justify-between items-center">
			<div class="flex flex-col space-y-3">
				<h2 class=" text-base font-bold uppercase text-gray-700 ">
					Director
				</h2>
				<p class=" font-semibold text-gray-600 text-xl">{{movieDoc.director}}</p>
			</div>
			<div class="flex flex-col space-y-3">
				<h2 class=" text-base font-bold uppercase text-gray-700 ">
					Release Date
				</h2>
				<p class=" font-semibold text-gray-600 text-xl">{{movieDoc.release_date}}</p>
			</div>
		</div>

		<div class="max-w-full ">

			<div class="mx-12 flex flex-col items-center" v-if="currentStep === 0">

				<div class="  mt-7 p-2 bg-white shadow-2xl content-center rounded">
					<img :src="movieDoc.poster" alt="Movie Poster">
				</div>

				<div class=" w-full flex items-center justify-center mt-7">
					<Button size="xl" variant="solid" @click="currentStep++">Book Tickets</Button>
				</div>

				<div class="flex flex-col space-y-3 mt-16">
					<h2 class=" text-base font-bold uppercase text-gray-700 ">
						Synopsis
					</h2>
					<p class=" font-normal text-gray-600 text-lg" >
						{{movieDoc.synopsis}}
					</p>
				</div>

			</div>
			

			<div v-else-if="currentStep === 1">
				<h2 class="font-medium text-xl text-gray-900 mt-7">Home Many Seats?</h2>
				<div class=" w-full flex flex-col items-center justify-center space-y-5 mt-6 ">
					<Button size="lg" 
						:variant=" index === bookingData.numberOfSeats? 'white' : 'subtle' "
						v-for="index in 8" class="w-full shadow-lg" 
						@click="setNumberOfSeats(index)"
					>{{ index }} </Button>
				</div>

			</div>

			<div v-else-if="currentStep === 2">
				<div class="flex flex-col space-y-4">
					<label class="font-medium text-xl text-gray-900 mt-7">Select Date</label>
					<Input class="" 
						type="date" 
						v-model="bookingData.date"
					/>
				</div>
			</div>

			<div v-else-if="currentStep === 3">
				<div class="flex flex-col space-y-4">
					<h3 class="font-medium text-xl text-gray-900 mt-7">Select Cinema & Show</h3>
				</div>

				<!-- Card Wrapper -->
				<div class=" space-y-2">
					<!-- Card Element -->
					<div 
						class="mt-6 bg-white shadow-xl p-4 rounded flex flex-col space-y-4"
						v-if="theatres.data && !theatres.loading"
						v-for="theatre in Object.keys(theatres.data)"
						:key="theatre"
					>
						<h2 class=" text-sm font-bold text-gray-800">{{theatre}}</h2>
						<div class="flex flex-row space-x-2" >
							<Button 
								size="sm" 
								class=" shadow-xl"
								:variant=" show.name === bookingData.show  ? 'outline' : 'subtle' " 
								:key = "show.name" 
								@click = "bookingData.show = show.name"
								v-for="show in theatres.data[theatre]"
							>
								{{show.start_time}}
							</Button>		
						</div>
					</div>

				
				</div>

			</div>
			
			<div v-else-if="currentStep === 4">
				<h3 class="font-medium text-xl text-gray-900 mt-7">Select Seats</h3>
				<div>
					<div class="flex flex-row align-middle gap-4 justify-center" v-for="row in Object.keys(seatStructure)" 
						:key="row"
					>

							<span 
								class="h-8 w-6 m-2 bg-blue-300 rounded-[2px]" 
								:class="[
									seat[1] === 'Available' 
										? 'bg-blue-300' 
										: seat[1] === 'Selected' 
											? 'bg-blue-600' 
											: 'bg-gray-300',
									hasSelectedCorrectNumberOfSeats 
										? 'cursor-not-allowed' 
										: 'cursor-pointer'
									]"
								
								@click="selectSeat(row, seat[0])"
								v-for="seat in seatStructure[row]
							">
							</span>
					</div>
				</div>
			</div>

			<div v-else class=" w-full flex items-center flex-col mt-7">
				<h1 class=" text-[150px]">🍿</h1>
				<h2 class=" font-medium text-xl text-gray-900 mt-7" >Enjoy The Movie</h2>
				<Button class=" mt-2" size="md" variant="solid" @click="currentStep = 0">Book More Tickets</Button>
			</div>

			<!-- Default Actions -->
			<div v-if="currentStep !== 0 && currentStep !== 5" class="flex flex-row mt-6 space-x-2">
				<Button size="lg" variant="subtle" @click="currentStep--" >Go Back</Button>
				<!-- nextButtonEnabled has 2 dependencies, currentStep and Booking Data
					so whenever these 2 changes the computed property is triggered
				-->
				<Button size="lg" variant="solid" 
					:disabled="!nextButtonEnabled"
					@click="moveToNextStep" 
				>
					Next
				</Button>
			</div>
		</div>

	</div>
</template>
<style></style>
