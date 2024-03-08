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

const currentWeather = ref();

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
						<UFormGroup class="p-2" label="Group Name" required>
							<UInput v-model="groupName" icon="i-ic-baseline-groups" />
						</UFormGroup>
						<UFormGroup class="p-2" label="# Adult Participants" required>
							<UInput v-model="adults" icon="ic:baseline-group" type="number" />
						</UFormGroup>
						<UFormGroup class="p-2" label="# Youth Participants" required>
							<UInput v-model="youths" icon="healthicons:child-program" type="number" />
						</UFormGroup>
					</div>
					<div class="flex">
						<UFormGroup class="p-2" label="Site ID" required>
							<UInput v-model="siteId" icon="bxs:been-here" />
						</UFormGroup>
						<UFormGroup class="p-2" label="Latitude" required>
							<UInput v-model="latitude" icon="ri:globe-fill" />
						</UFormGroup>
						<UFormGroup class="p-2" label="Longitude" required>
							<UInput v-model="longitude" icon="ri:globe-fill" />
						</UFormGroup>
					</div>
					<div class="inline-flex">
						<UFormGroup class="p-2" label="Date" required>
							<UInput v-model="date" icon="solar:calendar-bold" type="date" />
						</UFormGroup>
						<UFormGroup class="p-2" label="Start Time" required>
							<UInput v-model="date" icon="mdi:clock-outline" type="time" />
						</UFormGroup>
						<UFormGroup class="p-2" label="Total Volunteer Minutes" required>
							<UInput v-model="totalVolunteerMinutes" placeholder="Travel+Sampling" tuiype="number" />
						</UFormGroup>
						<UFormGroup class="p-2" label="Miles Driven" required>
							<UInput v-model="milesDriven" type="number" icon="fa-solid:car-side" />
						</UFormGroup>
					</div>
				</div>
				<div class="border-4 p-1 border-gray-900">
					<div class="flex">
						<UFormGroup class="p-2" label="Current Weather" required>
							<URadio
								v-for="option of currentWeatherOptions"
								:key="option.value"
								v-model="currentWeather"
								:label="option.label"
								:value="option.value"
								:icon="option.icon"
							/>
						</UFormGroup>
					</div>
				</div>
			</div>
			<div class="flex justify-end mt-6">
				<Button class="text-gray-900" variant="primary">Submit</Button>
			</div>
		</Form>
	</div>
</template>
