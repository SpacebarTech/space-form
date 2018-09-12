<template lang='pug'>
.space-form.easy-scroll
	.top-bar
		h1 {{ form.title }}
		.icon-close(@click='cancel')

	.form-body
		input-layout(
			:layout='form.layout'
			:fields='form.fields'
			:form-inputs='form.components'
			:errors='errors'

			:starting-data='initialData'

			@input='value = $event'
			v-model='value',
		)

	.submit
		.button(@click='submit') submit
</template>

<script>
import InputLayout from '@spacebartech/input-layout';

const iterate = ( obj, callback ) => {
	if ( Array.isArray( obj ) ) {
		obj.forEach( ( item, i ) => callback( item, i ) );
		return;
	}

	const keys = Object.keys( obj );
	keys.forEach( ( key ) => {
		const item = obj[key];

		callback( item, key );
	} );
};

const clone = ( obj ) => {
	const newObj = Array.isArray( obj ) ? [] : {};

	iterate( obj, ( value, key ) => {
		newObj[key] = typeof value === 'object' ? clone( value ) : value;
	} );

	return newObj;
};

const validate = ( data, fields ) => {
	let allValid = true;

	const fieldKeys  = Object.keys( fields );
	const validation = fieldKeys.reduce( ( allValidation, key ) => {
		const item  = data[key];
		const field = fields[key];

		const addedValidation = {};
		const valid = field.validate( item );

		if ( valid !== true ) {
			addedValidation[key] = valid;
		}

		allValid &= valid; // eslint-disable-line

		return Object.assign( allValidation, addedValidation );
	}, {} );

	return allValid || validation;
};

export default {
	name : 'space-form',

	props : {
		form : {
			required : true,
		},

		startingData : {
			default : () => ( {} )
		}
	},

	data : () => ( {
		value  : {},
		errors : {},
	} ),

	computed : {

		initialData() {
			return clone( this.startingData );
		},

	},

	watch : {

		startingData : {
			immediate : true,
			handler( data ) {
				this.value = clone( data );
			},
		},

	},

	mounted() {
	},

	methods : {

		cancel() {
			this.form.reject( 'user_cancelled' );
		},

		submit() {

			this.validate()
				.then( () => {
					this.valid = true;
					this.form.resolve( this.value );
				} )
				.catch( ( errors ) => {
					this.errors = errors;
				} );
		},

		validate() {
			this.errors = {};

			return new Promise( ( resolve, reject ) => {

				this.$nextTick( () => {
					const validation = validate( this.value, this.form.fields );

					if ( typeof validation === 'object' && Object.keys( validation ).length ) {
						reject( validation );

						return;
					}

					resolve( validation );
				} );
			} );
		}

	},

	components : {
		InputLayout
	}
};
</script>

<style lang='scss'>

.space-form {
	height: 100%;
	width: 100%;
	overflow: auto;
	background: transparent;
	padding-bottom: 40px;

	.top-bar {
		display: flex;
		padding: 25px;
		align-items: center;
		justify-content: space-between;

	}

	.submit {
		margin-top: 30px;
		display: flex;
		align-items: center;
		justify-content: center;
	}
}
</style>
