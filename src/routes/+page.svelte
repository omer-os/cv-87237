<script lang="ts">
	import { browser } from '$app/environment';

	const defaultCv = {
		personal: {
			fullName: 'Omar Chatin Abdulkareem',
			title: 'Frontend Engineer',
			email: 'omerchetin19@gmail.com',
			phone: '+964 771 069 7645',
			location: 'Kirkuk, Iraq',
			summary:
				'Frontend engineer with 4+ years of experience building mobile-first, responsive web applications at scale. Proficient in React, Next.js, and TypeScript with strong knowledge of state management, component architecture, and performance optimization. Experienced with Git, Webpack, Sass, and Node.js. Eager to bring expertise to Vue.js and contribute to high-traffic, user-facing products in a remote-first environment.',
			links: [
				{ label: 'LinkedIn', url: 'linkedin.com/in/omerchetin' },
				{ label: 'GitHub', url: 'github.com/omer-os' }
			]
		},
		sections: [
			{
				id: 'skills',
				title: 'Skills & Technologies',
				type: 'skills',
				items: [
					{
						category: 'Frameworks',
						values: ['React', 'Next.js', 'SvelteKit', 'TypeScript', 'JavaScript', 'ES6+']
					},
					{
						category: 'Styling & UI',
						values: ['TailwindCSS', 'SCSS', 'CSS3', 'shadcn/ui', 'Responsive Design', 'Mobile-First', 'RTL UI']
					},
					{
						category: 'State & Data',
						values: ['React Query', 'Zustand', 'Jotai', 'REST APIs', 'OpenAPI/Orval', 'Async Data Patterns']
					},
					{
						category: 'Build & DevOps',
						values: ['Vite', 'Webpack', 'Git', 'GitHub Actions', 'CI/CD', 'Railway', 'NPM']
					},
					{
						category: 'Backend & Tooling',
						values: ['Node.js', 'Bun', 'ElysiaJS', 'Prisma', 'PostgreSQL', 'Figma', 'Notion']
					}
				]
			},
			{
				id: 'experience',
				title: 'Work Experience',
				type: 'experience',
				items: [
					{
						title: 'Full Stack Developer',
						subtitle: 'Rasil — rasil.store',
						date: '2025',
						location: '',
						description: '',
						details: [
							'Built mobile-first, responsive frontends for a multi-tenant e-commerce SaaS platform serving Iraqi social media sellers',
							'Implemented SSR, ISR, and SEO optimization in Next.js with fully responsive, RTL-friendly layouts across 4 languages',
							'Developed typed backend with OpenAPI docs and auto-generated frontend API clients using Orval for consistent data contracts'
						]
					},
					{
						title: 'Founder & Lead Developer',
						subtitle: 'Kitly — moon.kitly.menu',
						date: '2023',
						location: '',
						description: '',
						details: [
							'Designed and shipped a mobile-first PWA with offline support, serving hundreds of daily users across 10+ restaurant businesses',
							'Built reusable component libraries with multi-tenant architecture, RBAC, and subscription management',
							'Delivered 4-language, RTL-friendly interfaces with responsive layouts optimized for performance on mobile devices'
						]
					},
					{
						title: 'Frontend Web Developer',
						subtitle: 'Qasah — qasah.ae',
						date: '2021 – Dec 2025',
						location: '',
						description: '',
						details: [
							'Developed responsive dashboards and data-heavy B2B SaaS interfaces, translating product requirements into production-ready UI',
							'Built complex forms and API-integrated frontend features with multilingual, RTL-friendly responsive layouts',
							'Participated in code reviews and collaborated with cross-functional engineering teams to maintain architecture standards'
						]
					}
				]
			},
			{
				id: 'projects',
				title: 'Featured Projects',
				type: 'projects',
				items: [
					{
						title: 'IQD Wiki',
						subtitle: 'iqdwiki.com · github.com/omer-os/iqd-wiki',
						date: '',
						location: '',
						description: 'Open-source community project helping Iraqi developers learn programming.',
						details: [
							'Contributed frontend and backend development with an issue submission workflow integrated to GitHub',
							'Automated notifications via Telegram API and GitHub Actions CI/CD pipelines'
						]
					}
				]
			},
			{
				id: 'education',
				title: 'Education',
				type: 'education',
				items: [
					{
						title: "Bachelor's Degree in Dental Technology",
						subtitle: 'Al-Kitab University',
						date: '2020 – 2024',
						location: 'Kirkuk, Iraq',
						description: '',
						details: []
					}
				]
			},
			{
				id: 'languages',
				title: 'Languages',
				type: 'languages',
				items: [
					{ name: 'Turkish', level: 'Native' },
					{ name: 'Arabic', level: 'Fluent' },
					{ name: 'English', level: 'Fluent' }
				]
			}
		]
	};

	let jsonText = $state(JSON.stringify(defaultCv, null, 2));
	let cv = $state(structuredClone(defaultCv));
	let showEditor = $state(false);
	let parseError = $state('');

	function applyJson() {
		try {
			cv = JSON.parse(jsonText);
			parseError = '';
		} catch (e: any) {
			parseError = e.message;
		}
	}

	function handleFileUpload(e: Event) {
		const file = (e.target as HTMLInputElement).files?.[0];
		if (!file) return;
		const reader = new FileReader();
		reader.onload = () => {
			jsonText = reader.result as string;
			applyJson();
		};
		reader.readAsText(file);
	}

	async function exportPdf() {
		if (!browser) return;
		const html2pdf = (await import('html2pdf.js')).default;
		const el = document.getElementById('cv-render');
		if (!el) return;
		html2pdf()
			.set({
				margin: 0,
				filename: `${cv.personal.fullName.replace(/\s+/g, '_')}_CV.pdf`,
				image: { type: 'jpeg', quality: 0.98 },
				html2canvas: { scale: 2, useCORS: true },
				jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
			})
			.from(el)
			.save();
	}
</script>

<!-- Toolbar -->
<div class="no-print sticky top-0 z-50 flex items-center gap-3 border-b border-neutral-200 bg-white px-5 py-3">
	<button
		onclick={() => (showEditor = !showEditor)}
		class="rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50"
	>
		{showEditor ? 'Hide Editor' : 'Edit JSON'}
	</button>
	<button
		onclick={exportPdf}
		class="rounded bg-neutral-900 px-3 py-1.5 text-xs font-medium text-white transition hover:bg-neutral-700"
	>
		Export PDF
	</button>
	<label class="cursor-pointer rounded border border-neutral-300 px-3 py-1.5 text-xs font-medium text-neutral-700 transition hover:bg-neutral-50">
		Load JSON
		<input type="file" accept=".json" onchange={handleFileUpload} class="hidden" />
	</label>
	{#if parseError}
		<span class="text-xs text-red-500">{parseError}</span>
	{/if}
</div>

{#if showEditor}
	<div class="no-print border-b border-neutral-200 bg-neutral-50 p-4">
		<textarea
			bind:value={jsonText}
			oninput={() => applyJson()}
			class="h-[40vh] w-full rounded border border-neutral-300 bg-white p-3 font-mono text-xs focus:border-neutral-500 focus:outline-none"
			spellcheck="false"
		></textarea>
	</div>
{/if}

<!-- Page background -->
<div class="flex justify-center bg-neutral-100 py-10 print:bg-white print:py-0">
	<div
		id="cv-render"
		class="w-[210mm] min-h-[297mm] bg-white shadow-sm print:shadow-none print:w-full"
		style="font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;"
	>
		<!-- Header -->
		<header class="border-b border-neutral-200 px-12 py-8">
			<div class="flex items-start justify-between">
				<div>
					<h1 class="text-[26px] font-bold tracking-tight text-neutral-900">
						{cv.personal.fullName}
					</h1>
					<p class="mt-0.5 text-sm font-medium text-neutral-500">{cv.personal.title}</p>
				</div>
				<div class="text-right text-[11px] leading-5 text-neutral-500 mt-0.5">
					{#if cv.personal.phone}<div>{cv.personal.phone}</div>{/if}
					{#if cv.personal.location}<div>{cv.personal.location}</div>{/if}
					{#if cv.personal.email}<div>{cv.personal.email}</div>{/if}
					{#each cv.personal.links as link}
						<div class="text-neutral-700 font-medium">{link.label}: {link.url}</div>
					{/each}
				</div>
			</div>
			{#if cv.personal.summary}
				<p class="mt-4 text-[11.5px] leading-relaxed text-neutral-600 max-w-[155mm]">
					{cv.personal.summary}
				</p>
			{/if}
		</header>

		<!-- Sections -->
		<main class="px-12 py-6 space-y-6">
			{#each cv.sections as section}
				<section>
					<h2 class="mb-2.5 text-[10px] font-bold uppercase tracking-[0.12em] text-neutral-400 border-b border-neutral-100 pb-1.5">
						{section.title}
					</h2>

					{#if section.type === 'skills'}
						<div class="space-y-1">
							{#each section.items as skill}
								<div class="flex gap-1.5 text-[11.5px]">
									<span class="w-[110px] shrink-0 font-medium text-neutral-700">{skill.category}</span>
									<span class="text-neutral-500">{skill.values.join(', ')}</span>
								</div>
							{/each}
						</div>

					{:else if section.type === 'experience' || section.type === 'education' || section.type === 'projects'}
						<div class="space-y-4">
							{#each section.items as item}
								<div>
									<div class="flex items-baseline justify-between gap-4">
										<div>
											<span class="text-[12px] font-semibold text-neutral-800">{item.title}</span>
											{#if item.subtitle}
												<span class="text-[11.5px] text-neutral-400 ml-1.5">· {item.subtitle}</span>
											{/if}
										</div>
										<div class="shrink-0 text-[11px] text-neutral-400">
											{#if item.date}{item.date}{/if}
											{#if item.location && item.date} · {/if}
											{#if item.location}{item.location}{/if}
										</div>
									</div>
									{#if item.description}
										<p class="mt-1 text-[11.5px] text-neutral-600">{item.description}</p>
									{/if}
									{#if item.details?.length}
										<ul class="mt-1.5 space-y-0.5 text-[11.5px] text-neutral-600 list-none">
											{#each item.details as detail}
												<li class="flex gap-2">
													<span class="mt-[5px] h-[3px] w-[3px] shrink-0 rounded-full bg-neutral-400"></span>
													<span>{detail}</span>
												</li>
											{/each}
										</ul>
									{/if}
								</div>
							{/each}
						</div>

					{:else if section.type === 'languages'}
						<div class="flex gap-6 text-[11.5px]">
							{#each section.items as lang}
								<div>
									<span class="font-medium text-neutral-700">{lang.name}</span>
									<span class="text-neutral-400 ml-1">({lang.level})</span>
								</div>
							{/each}
						</div>
					{/if}
				</section>
			{/each}
		</main>
	</div>
</div>
