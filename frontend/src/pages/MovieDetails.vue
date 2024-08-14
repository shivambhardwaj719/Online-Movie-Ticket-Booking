<template>
    <div v-if="!movieResource.loading && movieResource.doc">
        <!-- page-1 data -->
        <!-- Movie Name -->
        <h1 class="text-gray-900 font-bold text-[32px]" style="text-align: center;">{{ movieDoc.title }}</h1>

        <!-- Director Name -->
        <div class="mt-11 flex flex-row items-center justify-between">
            <div class="flex flex-col space-y-3">
                <h2 class="text-gray-700 font-bold uppercase">Director</h2>
                <p class="text-gray-600 semi-bold">{{movieDoc.director}}</p>
            </div>
            <!-- Released Date -->
            <div class="flex flex-col space-y-3">
                <h2 class="text-gray-700 font-bold uppercase">Released Date</h2>
                <p class="text-gray-600 semi-bold">{{ movieDoc.releasing_date }}</p>
            </div>
            <!--poster and synopsis -->
        </div>
        <div class="max-w-full">
            <div class="mx-10" v-if="currentStep===0">
                <div class="mt-4 bg-white shadow-xl rounded">
                    <img
                    :src="movieDoc.poster"
                    alt="Movie Poster">
                </div>

                <div class="w-full flex item-center justify-center mt-10">
                    <Button style="background-color: 
                        black; color: blue;" size="xl" variant="solid" 
                        @click="currentStep++">Book Ticket</Button>
                </div>

                <div class="flex flex-col space-y-3 mt-16">
                    <h2 class="text-gray-700 font-bold uppercase">Synopsis</h2>
                    <p class="text-gray-600 text-lg font-formal">{{movieDoc.synopsis}}</p>
                </div>
            </div>
        <!-- page 2 data -->
            <div v-else-if="currentStep===1">
                <h2 class="font-medium text-xl my-7 text-gray-900">How many seats</h2>

                <div class="flex flex-col w-full space-y-5 mt-6">
                    <Button
                        v-for="index in 8"
                        :key="index"
                        size="lg" 
                        class="shadow-lg"
                        :class="index === bookingData.numberOfSeats ? 'bg-green-600' : 'bg-white'"
                        @click="setNumberOfSeats(index)">
                        {{ index }}
                    </Button>
                </div>

            </div>

            <!-- page-3 data -->

            <div v-else-if="currentStep === 2">
                <div class="flex flex-col space-y-4">
                  <h2 class="font-medium text-xl mt-7 text-gray-900">Date</h2>
                  <Input type="date" v-model="bookingData.date" />
                </div>
        
                <div class="flex flex-col space-y-4">
                  <h2 class="font-medium text-xl mt-7 text-gray-900">Cinema & Show</h2>
        
                  <div class="space-y-2">
                    <div
                      v-for="theater in Object.keys(theaterShows.data)"
                      :key="theater.name"
                      class="bg-white shadow-xl p-4 rounded flex flex-col space-y-4"
                    >
                      <h3 class="text-sm font-bold text-gray-800">{{ theater }}</h3>
                      <div class="flex flex-row space-x-2">
                        <Button
                          @click="bookingData.show = show.name"
                          :key="show.name"
                          v-for="show in theaterShows.data[theater]"
                          size="sm"
                          :variant="
                            show.name === bookingData.show ? 'subtle' : 'outline'
                          "
                          >{{ show.start_time }}</Button
                        >
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            <!-- page-4 data (seat choice) -->
            <div v-else-if="currentStep===3">
                <h2 class="font-medium text-xl mt-7 text-gray-900">Select seats</h2>

                <div>
                    <div :key="row" class="flex flex-row " v-for="row in Object.keys(seatStructure)">                        
                    <span
                        @click="selectSeats(row, seat[0])"
                        v-for="seat in seatStructure[row]"
                        class="w-6 h-8 m-4 bg-blue-300 shadow-xl rounded-[7px]"
                        :class="[
                            seat[1]==='Available' 
                            ? 'bg-blue-300' 
                            : seat[1] === 'Selected' 
                            ? 'bg-green-600' : 
                            'bg-gray-300', 
                            hasSelectedCorrectNumberOfSeats
                            ? 'cursor-not-allowed'
                            :'cursor-pointer'
                            ]" ></span>
                    </div>
                </div>

            </div>
            <div v-else-if="currentStep === 4">
                <div class="w-full flex items-center flex-col mt-10"> 
                <h1 class="text-[210px]">🍿</h1>
                <h2 class="font-medium text-xl mt-7 text-gray-900">Enjoy the Movie</h2>
                </div>
            </div>
        </div>

        <!-- Next and go-back btn -->

        <div class=" flex flex-row mt-6 space-x-2">
            <Button
                class=" shadow-xl mt-5 "
                size="lg" 
                variant="subtle" 
                v-if="currentStep !== 0 && currentStep !== 4 && currentStep !== 3" 
                @click="currentStep--"> 
                Go Back 
            </Button>
            <Button class=" shadow-xl mt-5 "
                v-if="currentStep !== 0 && currentStep !== 3 && currentStep !== 4"
                size="lg" 
                :variant="solid"
                @click="currentStep++">Next
            </Button>
            <Button
                class=" shadow-xl mt-5 "
                size="lg" 
                variant="subtle" 
                v-if="currentStep === 3" 
                @click="handleNextClick()"> 
                Confirm Booking
            </Button>
        </div>

</div>
</template>


<script setup>
    import { ref , reactive, computed } from 'vue';
    import { createDocumentResource , createListResource } from 'frappe-ui';
//    fetching the movie details
    const movieName = ref("KGF-Chapter:1")

    const movieResource = createDocumentResource({
        doctype : 'Movie',
        name : "KGF-Chapter:1",
        fields: ['name', 'director','releasing_date'],
        onSuccess(doc){
            console.log(doc)
        },
        auto: true
    })
    // fetching the show details
    const theaterShows = createListResource({
        doctype : 'Theatre Show',
        fields : ['theater','start_time','name'],
        auto : true,
        transform : (shows) => {
            const groupedShows = {}
            for(const show of shows) {
                if (!groupedShows[show.theater]) {
                    groupedShows[show.theater] = []
                }
                groupedShows[show.theater].push(show)
            }
            return groupedShows
        }
    })

    // const showByTheater = computed(() =>{
    //     if (theaterShows.loading || !theaterShows.data) {
    //         return {}
    //     }
    // })
    
// structure of seat
    function getSeatStructure(alphabets, numbers){
        const structure = {}
        for(const alphabet of alphabets){
            structure[alphabet] = []
            for(const number of numbers){
                structure[alphabet].push([number,'Available'])
            }
        }
        return structure
    }

    // getting seat structure of seat
    const seatStructure = reactive(getSeatStructure(
        ['A','B','C','D','E'],
        [1, 2, 3, 4, 5, 6, 7]))
    
    const movieBooking = createListResource({doctype: 'Ticket Booking', insert : {
        onSuccess(){
            console.log('Booking Created')
            currentStep.value++
        }
    }})

    const currentStep = ref(0)

    // date 
    const today = new Date().toISOString().substr(0,10)

    // booking data
    const bookingData = reactive({
        numberOfSeats : 0,
        selectedSeats : [],
        date : today,
        show : null,
    })

    function setNumberOfSeats(n){
        bookingData.numberOfSeats = n
    }

    // function for select seat
    function selectSeats(row, number){
        if (hasSelectedCorrectNumberOfSeats.value){
            return
        }
        const seat = seatStructure[row].find((seat) => seat[0] === number)
        seat[1] = 'Selected'
        bookingData.selectedSeats.push(`${row}${number}`)
    }

    const hasSelectedCorrectNumberOfSeats = computed(() => {
        return bookingData.selectedSeats.length === bookingData.numberOfSeats
    })

    const movieDoc = computed(()=>movieResource.doc)

    const nextButtonEnabled = computed(() =>{
        if (currentStep.value === 1) {
            return bookingData.numberOfSeats
        }
        else if (currentStep === 2) {
            return bookingData.show && bookingData.show
        }
        else if (currentStep === 3) {
            return hasSelectedCorrectNumberOfSeats
        }
    })

    function handleNextClick(){
        if(currentStep.value!=3){
            currentStep.value++
            return
        }
        movieBooking.insert.submit({
            movie: movieName.value,
            date : bookingData.date,
            show : bookingData.show,
            selected_seats : JSON.stringify(bookingData.selectedSeats),
            number_Of_tickets : bookingData.numberOfSeats,
        })
    }
</script>