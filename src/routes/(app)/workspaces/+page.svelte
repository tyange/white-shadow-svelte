<script lang="ts">
	import { onMount } from 'svelte';
	import type { AxiosResponse } from 'axios';

	import type { ResponseData } from '@/types/ResponseData';
	import type { Company } from '@/types/Company';
	import { axiosClient } from '@/api/axiosClient';
	import { Suitability } from '@/constants/Suitability';
	import { PageOptions } from '@/constants/PageOptions';

	import CompanyForm from '@/components/CompanyForm.svelte';
	import Box from '@/components/ui/Box.svelte';
	import CompanyItem from '@/components/CompanyItem.svelte';
	import { createNumberArray } from '@/utils/createNumberArray';

	let companies: Company[] = $state([]);
	let totalPageCount = $state(0);
	let pageNum = $state(1);

	const fetchAllCompanies = async (pageNum: number) => {
		try {
			const res: AxiosResponse<
				ResponseData<{
					data: { companies: Company[]; total_item_count: number };
				}>
			> = await axiosClient('/companies/all', {
				params: {
					pageSize: PageOptions.PageSize,
					pageNum
				}
			});
			const data = res.data;

			if (!data.data.companies) {
				companies = [];
				totalPageCount = 0;
				pageNum = 1;
				return;
			}

			companies = data.data.companies;

			const itemsPerPage = 4;
			const totalItemCount = data.data.total_item_count;

			totalPageCount = Math.ceil(totalItemCount / itemsPerPage);
		} catch (err) {
			console.error(err);
		}
	};

	const createCompany = async (companyName: string) => {
		try {
			await axiosClient.post('/companies/create', {
				company_name: companyName
			});
			await fetchAllCompanies(1);
		} catch (err) {
			console.error(err);
		}
	};

	const editCompany = async (companyId: number, enteredCompanyName: string) => {
		try {
			await axiosClient.put(`/companies/${companyId}`, {
				company_name: enteredCompanyName
			});
			await fetchAllCompanies(1);
		} catch (err) {
			console.error(err);
		}
	};

	const deleteCompany = async (companyId: number) => {
		try {
			await axiosClient.delete(`/companies/${companyId}`);
			await fetchAllCompanies(1);
		} catch (err) {
			console.error(err);
		}
	};

	const setPageNum = (num: number) => {
		pageNum = num;
	};

	$effect(() => {
		fetchAllCompanies(pageNum);
	});
</script>

<div class="flex h-full w-full items-center justify-center">
	<div class="flex h-1/2 w-3/5 flex-col gap-5">
		<div>
			<CompanyForm {createCompany} />
		</div>
		<Box suitability={Suitability.full}>
			<div
				class="flex h-full w-full flex-col items-center justify-center gap-5"
			>
				<div class="w-full flex-1">
					{#if companies.length === 0}
						<p>저장된 일하는 곳이 없습니다.</p>
					{:else}
						<div class="h-full w-full">
							<ul class="flex w-full flex-col gap-5">
								{#each companies as cp}
									<CompanyItem company={cp} {editCompany} {deleteCompany} />
								{/each}
							</ul>
						</div>
					{/if}
				</div>
				<div class="join">
					{#each createNumberArray(totalPageCount) as num}
						<button
							class="btn-netural btn join-item"
							onclick={() => setPageNum(num + 1)}
						>
							{num + 1}
						</button>
					{/each}
				</div>
			</div>
		</Box>
	</div>
</div>
