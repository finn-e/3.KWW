<script setup lang="ts">
const { path, query } = useRoute();
const router = useRouter();

// Filters
const search: Ref<any> = ref(query?.search?.toString() ?? undefined);
const debouncedSearch = refDebounced(search as any, 500);
const status: Ref<any> = ref(query.status?.toString() ?? undefined);
const page = ref(1);
const rowsPerPage = ref(25);

const { data, pending, error, refresh } = await useAsyncData(
	path,
	() => {
		// Build the params object
		const params: object = Object.assign(
			{},
			unref(search) && {
				search: unref(search),
			},
			unref(status) && {
				filter: {
					status: {
						_eq: unref(status),
					},
				},
			},
			{
				limit: unref(rowsPerPage),
				page: unref(page),
			},
		);

		const invoices = useDirectus(
			readItems('os_invoices', {
				fields: [
					'*',
					{
						contact: ['id', 'first_name', 'last_name', 'email'],
					},
				],
				...params,
			}),
		);

		// Total count of invoices
		const count = useDirectus(
			readItems('os_invoices', {
				aggregate: {
					count: ['*'],
				},
				filter: {
					...(unref(status) && {
						status: {
							_eq: unref(status),
						},
					}),
				},
			}),
		);

		return Promise.all([invoices, count]);
	},
	{
		transform: ([data, count]) => {
			return {
				invoices: data,
				count: parseInt(count[0].count),
			};
		},
	},
);

const invoices = computed(() => data.value?.invoices ?? []);
const totalCount = computed(() => data.value?.count ?? 0);
const pageFrom = computed(() => (page.value - 1) * rowsPerPage.value + (invoices.value.length ? 1 : 0));
const pageTo = computed(() => Math.min(page.value * rowsPerPage.value, totalCount.value));

const currentWeatherOptions = [
	{
		value: 'sunny',
		label: 'Clear/Sunny',
		icon: 'wi-day-sunny',
	},
	{
		value: 'overcast',
		label: 'Overcast',
		icon: 'wi-day-cloudy',
	},
	{
		value: 'intermittentRain',
		label: 'Intermittent Rain',
		icon: 'wi-day-showers',
	},
	{
		value: 'steadyRain',
		label: 'Steady Rain',
		icon: 'wi-day-rain',
	},
	{
		value: 'heavyRain',
		label: 'Heavy Rain',
		icon: 'wi-day-thunderstorm',
	},
];

const rainfallOptions = [
	{
		value: '0',
		label: '0"',
	},
	{
		value: '1',
		label: '0.1"',
	},
	{
		value: '5',
		label: '0.5"',
	},
	{
		value: '10',
		label: '1"',
	},
	{
		value: '15',
		label: '1.5"',
	},
	{
		value: '20',
		label: '>1.5"',
	},
];

const waterColorOptions = [
	{
		value: 'clear',
		label: 'Clear',
	},
	{
		value: 'brown/muddy',
		label: 'Brown/Muddy',
	},
	{
		value: 'green',
		label: 'Green',
	},
	{
		value: 'white',
		label: 'White',
	},
	{
		value: 'gray',
		label: 'Gray',
	},
	{
		value: 'orange',
		label: 'Orange',
	},
];

const odorOptions = [
	{
		value: 'none',
		label: 'None',
	},
	{
		value: 'rotteneggs',
		label: 'Rotten Eggs',
	},
	{
		value: 'chlorine',
		label: 'Chlorine',
	},
	{
		value: 'rancid/sour',
		label: 'Rancid/Sour',
	},
	{
		value: 'gas/Petro',
		label: 'Gas/Petro',
	},
	{
		value: 'musty',
		label: 'Musty',
	},
	{
		value: 'sweet/fruity',
		label: 'Sweet/Fruity',
	},
	{
		value: 'sharp/pungent',
		label: 'Sharp/Pungent',
	},
];

const waterSurfaceOptions = [
	{
		value: 'none',
		label: 'None',
	},
	{
		value: 'oilSheen',
		label: 'Oil Sheen',
	},
	{
		value: 'algae',
		label: 'Algae',
	},
	{
		value: 'soapSuds',
		label: 'Soap Suds',
	},
	{
		value: 'sewage',
		label: 'Sewage',
	},
	{
		value: 'erosion',
		label: 'Erosion',
	},
];

const streamFlowVisualOptions = [
	{
		value: 'flood',
		label: 'Flood',
	},
	{
		value: 'Bankfull',
		label: 'Bankfull',
	},
	{
		value: 'normal',
		label: 'Normal',
	},
	{
		value: 'low',
		label: 'Low',
	},
	{
		value: 'ponded',
		label: 'Ponded',
	},
	{
		value: 'dry',
		label: 'Dry',
	},
];

const currentWeather = ref();
const rainfall = ref();
const waterColor = ref();
const odor = ref();
const waterSurface = ref();
const streamFlowVisual = ref();

// Watch the search query and update the URL
watch([debouncedSearch, status, page], ([search, status, page]) => {
	router.push({
		path,
		query: {
			search: search?.toString(),
			status: status?.toString(),
			page: page?.toString(),
		},
	});

	refresh();
});
</script>
<template>
	<div>
		<PortalPageHeader
			title="New Sample"
			:breadcrumbs="[
				{
					title: 'Portal',
					href: '/portal',
				},
				{
					title: 'Samples',
					href: '/portal/samples',
				},
			]"
		></PortalPageHeader>
		<h1 class="text-2xl text-center text-gray-900">Kentucky Watershed Watch Monitoring Data Form</h1>

		<Form>
			<div class="relative items-start">
				<div class="border-4 p-1 border-gray-900">
					<div class="flex">
						<UFormGroup class="p-2 basis-1/3" label="Group Name" required>
							<UInput v-model="groupName" icon="i-ic-baseline-groups" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="# Adult Participants">
							<UInput v-model="adults" icon="ic:baseline-group" type="number" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="# Youth Participants">
							<UInput v-model="youths" icon="healthicons:child-program" type="number" />
						</UFormGroup>
					</div>
					<div class="flex">
						<UFormGroup class="p-2 basis-1/3" label="Site ID" required>
							<UInput v-model="siteId" icon="bxs:been-here" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="Latitude" required>
							<UInput v-model="latitude" icon="ri:globe-fill" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="Longitude" required>
							<UInput v-model="longitude" icon="ri:globe-fill" />
						</UFormGroup>
					</div>
					<div class="flex">
						<UFormGroup class="p-2 basis-1/4" label="Date" required>
							<UInput v-model="date" icon="solar:calendar-bold" type="date" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/4" label="Start Time" required>
							<UInput v-model="startTime" icon="mdi:clock-outline" type="time" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/4" label="Total Volunteer Minutes" required>
							<UInput v-model="totalVolunteerMinutes" placeholder="Travel+Sampling" tuiype="number" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/4" label="Miles Driven">
							<UInput v-model="milesDriven" type="number" icon="fa-solid:car-side" />
						</UFormGroup>
					</div>
				</div>
				<div class="border-4 p-1 border-gray-900">
					<div class="flex">
						<UFormGroup class="p-2" label="Current Weather">
							<URadio
								v-for="option of currentWeatherOptions"
								:key="option.value"
								v-model="currentWeather"
								:label="option.label"
								:value="option.value"
								:icon="option.icon"
							/>
						</UFormGroup>
						<UFormGroup class="p-2" label="Rainfall in last 48 hours (round up)">
							<URadio
								v-for="option of rainfallOptions"
								:key="option.value"
								v-model="rainfall"
								:label="option.label"
								:value="option.value"
							/>
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="Water Color">
							<URadio
								v-for="option of waterColorOptions"
								:key="option.value"
								v-model="waterColor"
								:label="option.label"
								:value="option.value"
							/>
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="Odor">
							<URadioGroup v-model="odor" :options="odorOptions" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/3" label="Water Surface">
							<URadioGroup v-model="waterSurface" :options="waterSurfaceOptions" />
						</UFormGroup>
					</div>
				</div>
				<div class="flex">
					<div class="border-4 p-1 border-gray-900 w-1/2">
						<UFormGroup class="p-2 basis-1/2" label="Stream Flow (Visual)">
							<URadioGroup v-model="streamFlowVisual" :options="streamFlowVisualOptions" />
						</UFormGroup>
						<UFormGroup class="p-2 basis-1/2" label="Stream Flow (Measured Cubic Ft/Sec)">
							<UInput v-model="streamFlowMeasured" icon="mdi:water" />
						</UFormGroup>
					</div>
					<div class="border-4 p-1 border-gray-900 w-1/2">
						<img src="assets/form_icons/stream_flow.png" alt="Stream Flow" class="w-full" />
					</div>
				</div>
			</div>
			<div class="flex justify-end mt-6">
				<Button class="text-gray-900" variant="primary">Submit</Button>
			</div>
		</Form>
	</div>
</template>
