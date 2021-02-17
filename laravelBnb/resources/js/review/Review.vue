<template>
    <div>
        <fatal-error v-if="true"></fatal-error>
        <div clss="row" v-else>
            <div :class="[{'col-md-4': twoColumns}, {'d-none': oneColumns}]">
            <div class="card">
                <div class="card-body">
                    <div v-if="loading">Loading...</div>
                    <div v-if="hasBooking">
                        <p>
                            Stayed at
                            <router-link :to="{name: 'bookable', params: { id: booking.bookable.bookable_id}}">{{ booking.bookable.title }}</router-link>
                        </p>
                        <p>From {{ booking.from }} to {{ booking.to }}</p>
                    </div>
                </div>
            </div>
        </div>
        <div :class="[{'col-md-8': twoColumns}, {'col-md-12': oneColumns}]">
            <div v-if="loading">Loading...</div>
            <div v-else>
                <div v-if="alreadyReviewed">
                    <h3>You've already left a review for this booking!</h3>
                </div>
                <div v-else>
                    <div class="form-group">
                        <label for="text-muted">Select the star rating (1 is worst - 5 is best)</label>
                            <star-rating class="fa-3x" v-model="review.rating"></star-rating>
                    </div>
                    <div class="form-group">
                        <label for="content" class="text-muted">Describe your experience with</label>
                            <textarea name="content" cols="30" rows="10" class="form-control" v-model="review.content" :class="[{'is-invalid': errorFor('content')}]"></textarea>
                            <v-errors :errors="errorFor('content')"></v-errors>
                    </div>
                        <button class="btn btn-lg btn-primary btn-block"
                        @click.prevent ="submit"
                        :disabled="sending"
                        >Submit</button>
                </div>
            </div>
        </div>
        </div>
    </div>
</template>


<script>
import {is404, is422} from "./../shared/utils/response";

export default {
    data() {
        return {
            review: {
                id: null,
                rating: 5,
                content: null
            },
            existingReview: null,
            loading: false,
            booking: null,
            error: false,
            errors: null,
            sending: false
        };
    },
    // methods: {
    //     onRatingChanged(rating) {
    //         console.log(rating);
    //     }
    // }
    created() {
        this.review.id = this.$route.params.id;
        this.sending = true;
        // 1. If review already exists (in reviews table by id)
        axios
        .get(`/api/reviews/${this.review.id}`)
        .then(response => {
            this.existingReview = response.data.data;
        })
        .catch(err => {
            if(is404(err)) {
                return axios
                .get(`/api/booking-by-review/${this.review.id}`)
                .then(response => {
                    this.booking = response.data.data;
                })
                .catch(err => {
                    //is404(err) ? {} : (this.error = true);
                    this.error = !is404(err);
                //     if(!is404(err)) {
                //     this.error =true;
                // }
                });
            }
            this.error = true;
        })
        .then(() => {
            //console.log(this.booking.booking_id);
            //console.log(response);
            this.sending = false;

            });
        // 2. Fetch a booking by a review key

    },
    computed: {
        alreadyReviewed() {
            return this.hasReview || !this.booking;
        },
        hasReview() {
            return this.existingReview !== null;
        },
        hasBooking() {
            return this.booking !== null;
        },
        oneColumns() {
            return !this.loading && this.alreadyReviewed;
        },
        twoColumns() {
            return this.loading || !this.alreadyReviewed;
        }

    },
    methods: {
        submit() {
             // 3. Store the review
             this.errors = null;
            this.loading = true;


            axios
            .post('/api/reviews',this.review)
            .then(response => console.log(response))
            .catch(err => {
                if (is422(err)) {
                    const errors = err.response.data.errors;

                    if (errors["content"] && 1 == _.size(errors)) {
                        this.errors = errors;
                        return;
                    }
                }

                this.error = true;
            })
            .then(() => (this.loading = false));
        },
        errorFor(field) {
            return null !== this.errors && this.errors[field] ? this.errors[field] : null;
            }
    }
};
</script>



