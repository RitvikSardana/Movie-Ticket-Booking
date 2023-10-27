<script setup>
import { Input } from 'frappe-ui';
import { Card } from 'frappe-ui';
import { Button } from 'frappe-ui';
import { reactive, ref } from 'vue';

const currentStep = ref(4)

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
	seatStructure[row][seat - 1][1] = "Selected"

	bookingData.selectedSeats.push([row, seat])
	console.log(bookingData.selectedSeats)
}



const bookingData = reactive({
	numberOfSeats:0,
	date: new Date().toISOString().slice(0,10),
	seats:seatStructure,
	selectedSeats: []
})


const setNumberOfSeats = (index) =>{
	bookingData.numberOfSeats = index
}





</script>

<template>
	<div>
		<h1 class=" text-gray-900 font-bold text-[32px] ">Oppenheimer</h1>

		<div class=" mt-11 flex justify-between items-center">
			<div class="flex flex-col space-y-3">
				<h2 class=" text-base font-bold uppercase text-gray-700 ">
					Director
				</h2>
				<p class=" font-semibold text-gray-600 text-xl">Christophar Nolan</p>
			</div>
			<div class="flex flex-col space-y-3">
				<h2 class=" text-base font-bold uppercase text-gray-700 ">
					Release Date
				</h2>
				<p class=" font-semibold text-gray-600 text-xl">21st July, 2023</p>
			</div>
		</div>

		<div class="max-w-full ">

			<div class="mx-12" v-if="currentStep === 0">

				<div class="  mt-7 p-2 bg-white shadow-2xl content-center rounded">
					<img src="https://pbs.twimg.com/media/FvUVt3hXgAAxP1H?format=jpg&name=900x900" alt="">
				</div>

				<div class=" w-full flex items-center justify-center mt-7">
					<Button size="xl" variant="solid" @click="currentStep++">Book Tickets</Button>
				</div>

				<div class="flex flex-col space-y-3 mt-16">
					<h2 class=" text-base font-bold uppercase text-gray-700 ">
						Synopsis
					</h2>
					<p class=" font-normal text-gray-600 text-lg ">
						During World War II, Lt. Gen. Leslie Groves Jr. appoints physicist J. Robert Oppenheimer to work on
						the
						top-secret Manhattan Project. Oppenheimer and a team of scientists spend years developing and
						designing
						the atomic bomb.
					</p>
				</div>

			</div>


			<div v-else-if="currentStep === 1">
				<h2 class="font-medium text-xl text-gray-900 mt-7">Home Many Seats?</h2>
				<div class=" w-full flex flex-col items-center justify-center space-y-5 mt-6 ">
					<Button size="lg" 
						:variant=" index === bookingData.numberOfSeats? 'subtle' : 'white' "
						v-for="index in 8" class="w-full shadow-lg" 
						@click="setNumberOfSeats(index)"
					>{{ index }} </Button>
				</div>

			</div>

			<div v-else-if="currentStep === 2">
				{{ bookingData.numberOfSeats  }}
				<div class="flex flex-col space-y-4">
					<label class="font-medium text-xl text-gray-900 mt-7">Home Many Seats?</label>
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
					<div class="mt-6 bg-white shadow-xl p-4 rounded flex flex-col space-y-4">
						<h2 class=" text-sm font-bold text-gray-800">Star Talkies</h2>
						<div class="flex flex-row space-x-2">
							<Button size="sm" variant="outline" class=" shadow-xl" >10:00 AM</Button>		
							<Button size="sm" variant="subtle" class=" shadow-xl" >13:00 PM</Button>		
						</div>
					</div>

					<!-- Card Element -->
					<div class="mt-6 bg-white shadow-xl p-4 rounded flex flex-col space-y-4">
						<h2 class=" text-sm font-bold text-gray-800">Anand Theatre</h2>
						<div class="flex flex-row space-x-2">
							<Button size="sm" variant="outline" class=" shadow-xl" >10:00 AM</Button>		
							<Button size="sm" variant="subtle" class=" shadow-xl" >13:00 PM</Button>		
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
								class="cursor-pointer h-8 w-6 m-2 bg-blue-300 rounded-[2px]" 
								:class="seat[1] === 'Available' ? 'bg-blue-300' 
									: seat[1] === 'Selected'? 'bg-blue-600' : 'bg-gray-300'  "
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

			</div>

			<!-- Default Actions -->
			<div v-if="currentStep !== 0 && currentStep !== 5" class="flex flex-row mt-6 space-x-2">
				<Button size="lg" variant="subtle" @click="currentStep--" >Go Back</Button>
				<Button size="lg" variant="solid" @click="currentStep++" >Next</Button>
			</div>





		</div>

	</div>
</template>
<style></style>
