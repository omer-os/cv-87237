<script lang="ts">
	import { browser } from '$app/environment';

	const defaultCv = {
		meta: {
			direction: 'ltr' as 'ltr' | 'rtl',
			primaryColor: '#1e40af',
			secondaryColor: '#334155',
			accentColor: '#3b82f6',
			bgColor: '#ffffff',
			textColor: '#1e293b',
			headerTextColor: '#ffffff',
			fontFamily: 'system-ui, sans-serif',
			layout: 'sidebar' as 'single' | 'sidebar',
			sidebarBg: '#1e293b',
			sidebarText: '#e2e8f0'
		},
		personal: {
			fullName: 'Omar Ahmed',
			title: 'Senior Frontend Developer',
			email: 'omar@example.com',
			phone: '+964 770 123 4567',
			location: 'Kirkuk, Iraq',
			website: 'https://omar.dev',
			summary:
				'Frontend developer with 4+ years of experience building production web applications using React, SvelteKit, and TypeScript. Passionate about creating elegant user interfaces and scalable SaaS products.',
			photo: '',
			links: [
				{ label: 'GitHub', url: 'https://github.com/omar' },
				{ label: 'LinkedIn', url: 'https://linkedin.com/in/omar' }
			]
		},
		sections: [
			{
				id: 'experience',
				title: 'Work Experience',
				type: 'experience',
				sidebar: false,
				items: [
					{
						title: 'Frontend Developer',
						subtitle: 'Qasah',
						date: '2023 - Present',
						location: 'Kirkuk, Iraq',
						description: '',
						details: [
							'Built and maintained multiple client-facing dashboards using React and TypeScript',
							'Implemented responsive designs with TailwindCSS achieving 99% mobile compatibility',
							'Collaborated with backend teams to integrate REST APIs and real-time data'
						]
					},
					{
						title: 'Freelance Web Developer',
						subtitle: 'Self-employed',
						date: '2021 - 2023',
						location: 'Remote',
						description: '',
						details: [
							'Delivered 10+ web projects for local businesses',
							'Built e-commerce solutions with payment integration',
							'Developed custom admin panels and CMS solutions'
						]
					}
				]
			},
			{
				id: 'education',
				title: 'Education',
				type: 'education',
				sidebar: false,
				items: [
					{
						title: 'Dental Technology',
						subtitle: 'Al-Kitab University',
						date: '2020 - 2024',
						location: 'Kirkuk, Iraq',
						description: '',
						details: []
					}
				]
			},
			{
				id: 'projects',
				title: 'Projects',
				type: 'projects',
				sidebar: false,
				items: [
					{
						title: 'Rasil',
						subtitle: 'Multi-tenant E-commerce SaaS',
						date: '2024',
						location: '',
						description:
							'SaaS platform for Iraqi Instagram sellers with subdomains, Arabic/English support, AI theme generation, and WhatsApp integration.',
						details: ['SvelteKit, TypeScript, PostgreSQL, Prisma, TailwindCSS']
					},
					{
						title: 'Kitly',
						subtitle: 'Restaurant Inventory SaaS',
						date: '2024',
						location: '',
						description: 'Multi-tenant restaurant and inventory management system.',
						details: ['React, Node.js, PostgreSQL, Prisma']
					}
				]
			},
			{
				id: 'skills',
				title: 'Skills',
				type: 'skills',
				sidebar: true,
				items: [
					{ category: 'Frontend', values: ['React', 'SvelteKit', 'Next.js', 'TypeScript', 'TailwindCSS'] },
					{ category: 'Backend', values: ['Node.js', 'Bun', 'Elysia.js', 'Prisma', 'PostgreSQL'] },
					{ category: 'Mobile', values: ['React Native', 'Expo'] },
					{ category: 'Tools', values: ['Git', 'Docker', 'Linux', 'Neovim'] }
				]
			},
			{
				id: 'languages',
				title: 'Languages',
				type: 'languages',
				sidebar: true,
				items: [
					{ name: 'Arabic', level: 'Native' },
					{ name: 'English', level: 'Fluent' },
					{ name: 'Turkish', level: 'Fluent' }
				]
			},
			{
				id: 'certifications',
				title: 'Certifications',
				type: 'certifications',
				sidebar: true,
				items: [{ title: 'AWS Cloud Practitioner', issuer: 'Amazon', date: '2024' }]
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

	let mainSections = $derived(cv.sections.filter((s: any) => !s.sidebar));
	let sidebarSections = $derived(cv.sections.filter((s: any) => s.sidebar));
	let isSidebar = $derived(cv.meta.layout === 'sidebar');
	let isRtl = $derived(cv.meta.direction === 'rtl');
</script>

<div class="no-print sticky top-0 z-50 flex flex-wrap items-center gap-3 border-b border-gray-200 bg-white px-4 py-3 shadow-sm">
	<button
		onclick={() => (showEditor = !showEditor)}
		class="rounded-lg bg-gray-800 px-4 py-2 text-sm font-medium text-white transition hover:bg-gray-700"
	>
		{showEditor ? 'Hide Editor' : 'Edit JSON'}
	</button>
	<button
		onclick={exportPdf}
		class="rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white transition hover:bg-blue-700"
	>
		Export PDF
	</button>
	<label
		class="cursor-pointer rounded-lg border border-gray-300 px-4 py-2 text-sm font-medium transition hover:bg-gray-50"
	>
		Load JSON File
		<input type="file" accept=".json" onchange={handleFileUpload} class="hidden" />
	</label>
	{#if parseError}
		<span class="text-sm text-red-500">{parseError}</span>
	{/if}
</div>

{#if showEditor}
	<div class="no-print border-b border-gray-200 bg-gray-50 p-4">
		<textarea
			bind:value={jsonText}
			oninput={() => applyJson()}
			class="h-[40vh] w-full rounded-lg border border-gray-300 bg-white p-3 font-mono text-xs focus:border-blue-500 focus:ring-1 focus:ring-blue-500 focus:outline-none"
			spellcheck="false"
		></textarea>
	</div>
{/if}

<div class="flex justify-center bg-gray-100 p-8 print:bg-white print:p-0">
	<div
		id="cv-render"
		dir={cv.meta.direction}
		style="
			--primary: {cv.meta.primaryColor};
			--secondary: {cv.meta.secondaryColor};
			--accent: {cv.meta.accentColor};
			--bg: {cv.meta.bgColor};
			--text: {cv.meta.textColor};
			--header-text: {cv.meta.headerTextColor};
			--sidebar-bg: {cv.meta.sidebarBg};
			--sidebar-text: {cv.meta.sidebarText};
			font-family: {cv.meta.fontFamily};
			color: {cv.meta.textColor};
			background: {cv.meta.bgColor};
		"
		class="w-[210mm] min-h-[297mm] shadow-xl print:shadow-none print:w-full"
	>
		{#if isSidebar}
			<div class="flex min-h-[297mm]" class:flex-row-reverse={isRtl}>
				<div
					class="w-[70mm] shrink-0 p-6"
					style="background: {cv.meta.sidebarBg}; color: {cv.meta.sidebarText};"
				>
					{#if cv.personal.photo}
						<div class="mb-4 flex justify-center">
							<img
								src={cv.personal.photo}
								alt={cv.personal.fullName}
								class="h-28 w-28 rounded-full border-4 object-cover"
								style="border-color: {cv.meta.accentColor};"
							/>
						</div>
					{/if}

					<div class="mb-6">
						<h2 class="mb-3 text-xs font-bold tracking-widest uppercase opacity-60">Contact</h2>
						<div class="space-y-2 text-sm">
							{#if cv.personal.email}
								<div class="flex items-start gap-2" class:flex-row-reverse={isRtl}>
									<span class="opacity-50">✉</span>
									<span class="break-all">{cv.personal.email}</span>
								</div>
							{/if}
							{#if cv.personal.phone}
								<div class="flex items-start gap-2" class:flex-row-reverse={isRtl}>
									<span class="opacity-50">☎</span>
									<span>{cv.personal.phone}</span>
								</div>
							{/if}
							{#if cv.personal.location}
								<div class="flex items-start gap-2" class:flex-row-reverse={isRtl}>
									<span class="opacity-50">⌖</span>
									<span>{cv.personal.location}</span>
								</div>
							{/if}
							{#if cv.personal.website}
								<div class="flex items-start gap-2" class:flex-row-reverse={isRtl}>
									<span class="opacity-50">⊕</span>
									<span class="break-all">{cv.personal.website}</span>
								</div>
							{/if}
						</div>
					</div>

					{#if cv.personal.links.length > 0}
						<div class="mb-6">
							<h2 class="mb-3 text-xs font-bold tracking-widest uppercase opacity-60">Links</h2>
							<div class="space-y-1.5 text-sm">
								{#each cv.personal.links as link}
									<div class="break-all opacity-90">{link.label}: {link.url}</div>
								{/each}
							</div>
						</div>
					{/if}

					{#each sidebarSections as section}
						<div class="mb-6">
							<h2 class="mb-3 text-xs font-bold tracking-widest uppercase opacity-60">
								{section.title}
							</h2>
							{#if section.type === 'skills'}
								{#each section.items as skill}
									<div class="mb-3">
										<div class="mb-1 text-xs font-semibold uppercase opacity-70">
											{skill.category}
										</div>
										<div class="flex flex-wrap gap-1.5">
											{#each skill.values as v}
												<span
													class="rounded px-2 py-0.5 text-xs"
													style="background: {cv.meta.accentColor}22; color: {cv.meta.sidebarText};"
												>
													{v}
												</span>
											{/each}
										</div>
									</div>
								{/each}
							{:else if section.type === 'languages'}
								<div class="space-y-1.5 text-sm">
									{#each section.items as lang}
										<div class="flex justify-between">
											<span>{lang.name}</span>
											<span class="opacity-60">{lang.level}</span>
										</div>
									{/each}
								</div>
							{:else if section.type === 'certifications'}
								<div class="space-y-2 text-sm">
									{#each section.items as cert}
										<div>
											<div class="font-medium">{cert.title}</div>
											<div class="text-xs opacity-60">{cert.issuer} · {cert.date}</div>
										</div>
									{/each}
								</div>
							{:else}
								{#each section.items as item}
									<div class="mb-2 text-sm">
										{#if item.title}<div class="font-medium">{item.title}</div>{/if}
										{#if item.description}<div class="opacity-80">{item.description}</div>{/if}
									</div>
								{/each}
							{/if}
						</div>
					{/each}
				</div>

				<div class="flex-1 p-8">
					<header class="mb-6">
						<h1 class="text-3xl font-bold" style="color: {cv.meta.primaryColor};">
							{cv.personal.fullName}
						</h1>
						<div class="mt-1 text-lg font-medium" style="color: {cv.meta.secondaryColor};">
							{cv.personal.title}
						</div>
						{#if cv.personal.summary}
							<p class="mt-3 text-sm leading-relaxed opacity-80">{cv.personal.summary}</p>
						{/if}
					</header>

					{#each mainSections as section}
						<div class="mb-6">
							<h2
								class="mb-3 border-b-2 pb-1 text-sm font-bold tracking-widest uppercase"
								style="border-color: {cv.meta.primaryColor}; color: {cv.meta.primaryColor};"
							>
								{section.title}
							</h2>
							{#if section.type === 'experience' || section.type === 'education' || section.type === 'projects'}
								{#each section.items as item, i}
									<div class="mb-4" class:mb-0={i === section.items.length - 1}>
										<div class="flex items-start justify-between gap-4" class:flex-row-reverse={isRtl}>
											<div>
												<div class="font-semibold" style="color: {cv.meta.secondaryColor};">
													{item.title}
												</div>
												{#if item.subtitle}
													<div class="text-sm" style="color: {cv.meta.accentColor};">
														{item.subtitle}
													</div>
												{/if}
											</div>
											<div class="shrink-0 text-xs opacity-60" class:text-start={isRtl} class:text-end={!isRtl}>
												{#if item.date}<div>{item.date}</div>{/if}
												{#if item.location}<div>{item.location}</div>{/if}
											</div>
										</div>
										{#if item.description}
											<p class="mt-1 text-sm opacity-80">{item.description}</p>
										{/if}
										{#if item.details?.length}
											<ul
												class="mt-1.5 space-y-0.5 text-sm"
												class:list-disc={!isRtl}
												style={isRtl ? '' : 'padding-inline-start: 1.25rem;'}
											>
												{#each item.details as detail}
													<li>
														{#if isRtl}<span class="me-1">•</span>{/if}{detail}
													</li>
												{/each}
											</ul>
										{/if}
									</div>
								{/each}
							{:else if section.type === 'skills'}
								{#each section.items as skill}
									<div class="mb-2">
										<span class="text-sm font-semibold">{skill.category}: </span>
										<span class="text-sm opacity-80">{skill.values.join(', ')}</span>
									</div>
								{/each}
							{:else if section.type === 'languages'}
								<div class="flex flex-wrap gap-4 text-sm">
									{#each section.items as lang}
										<span>
											<span class="font-medium">{lang.name}</span>
											<span class="opacity-60">({lang.level})</span>
										</span>
									{/each}
								</div>
							{:else if section.type === 'certifications'}
								{#each section.items as cert}
									<div class="mb-1 text-sm">
										<span class="font-medium">{cert.title}</span>
										<span class="opacity-60"> — {cert.issuer}, {cert.date}</span>
									</div>
								{/each}
							{:else}
								{#each section.items as item}
									<div class="mb-2 text-sm">
										{#if item.title}<div class="font-semibold">{item.title}</div>{/if}
										{#if item.subtitle}<div class="opacity-60">{item.subtitle}</div>{/if}
										{#if item.description}<p class="opacity-80">{item.description}</p>{/if}
										{#if item.details?.length}
											<ul class="mt-1 space-y-0.5" style="padding-inline-start: 1.25rem;">
												{#each item.details as d}
													<li>{d}</li>
												{/each}
											</ul>
										{/if}
									</div>
								{/each}
							{/if}
						</div>
					{/each}
				</div>
			</div>
		{:else}
			<div class="p-10">
				<header class="mb-6 text-center">
					{#if cv.personal.photo}
						<div class="mb-3 flex justify-center">
							<img
								src={cv.personal.photo}
								alt={cv.personal.fullName}
								class="h-24 w-24 rounded-full border-4 object-cover"
								style="border-color: {cv.meta.primaryColor};"
							/>
						</div>
					{/if}
					<h1 class="text-3xl font-bold" style="color: {cv.meta.primaryColor};">
						{cv.personal.fullName}
					</h1>
					<div class="mt-1 text-lg" style="color: {cv.meta.secondaryColor};">
						{cv.personal.title}
					</div>
					<div class="mt-2 flex flex-wrap justify-center gap-3 text-sm opacity-70">
						{#if cv.personal.email}<span>{cv.personal.email}</span>{/if}
						{#if cv.personal.phone}<span>· {cv.personal.phone}</span>{/if}
						{#if cv.personal.location}<span>· {cv.personal.location}</span>{/if}
						{#if cv.personal.website}<span>· {cv.personal.website}</span>{/if}
					</div>
					{#if cv.personal.links.length > 0}
						<div class="mt-1 flex flex-wrap justify-center gap-3 text-sm">
							{#each cv.personal.links as link}
								<span style="color: {cv.meta.accentColor};">{link.label}: {link.url}</span>
							{/each}
						</div>
					{/if}
					{#if cv.personal.summary}
						<p class="mx-auto mt-4 max-w-2xl text-sm leading-relaxed opacity-80">
							{cv.personal.summary}
						</p>
					{/if}
				</header>

				{#each cv.sections as section}
					<div class="mb-5">
						<h2
							class="mb-3 border-b-2 pb-1 text-sm font-bold tracking-widest uppercase"
							style="border-color: {cv.meta.primaryColor}; color: {cv.meta.primaryColor};"
						>
							{section.title}
						</h2>
						{#if section.type === 'experience' || section.type === 'education' || section.type === 'projects'}
							{#each section.items as item, i}
								<div class="mb-4" class:mb-0={i === section.items.length - 1}>
									<div class="flex items-start justify-between gap-4" class:flex-row-reverse={isRtl}>
										<div>
											<div class="font-semibold" style="color: {cv.meta.secondaryColor};">
												{item.title}
											</div>
											{#if item.subtitle}
												<div class="text-sm" style="color: {cv.meta.accentColor};">
													{item.subtitle}
												</div>
											{/if}
										</div>
										<div class="shrink-0 text-xs opacity-60">
											{#if item.date}<div>{item.date}</div>{/if}
											{#if item.location}<div>{item.location}</div>{/if}
										</div>
									</div>
									{#if item.description}
										<p class="mt-1 text-sm opacity-80">{item.description}</p>
									{/if}
									{#if item.details?.length}
										<ul class="mt-1.5 list-disc space-y-0.5 text-sm" style="padding-inline-start: 1.25rem;">
											{#each item.details as detail}
												<li>{detail}</li>
											{/each}
										</ul>
									{/if}
								</div>
							{/each}
						{:else if section.type === 'skills'}
							{#each section.items as skill}
								<div class="mb-2">
									<span class="text-sm font-semibold">{skill.category}: </span>
									<span class="text-sm opacity-80">{skill.values.join(', ')}</span>
								</div>
							{/each}
						{:else if section.type === 'languages'}
							<div class="flex flex-wrap gap-4 text-sm">
								{#each section.items as lang}
									<span>
										<span class="font-medium">{lang.name}</span>
										<span class="opacity-60">({lang.level})</span>
									</span>
								{/each}
							</div>
						{:else if section.type === 'certifications'}
							{#each section.items as cert}
								<div class="mb-1 text-sm">
									<span class="font-medium">{cert.title}</span>
									<span class="opacity-60"> — {cert.issuer}, {cert.date}</span>
								</div>
							{/each}
						{:else}
							{#each section.items as item}
								<div class="mb-2 text-sm">
									{#if item.title}<div class="font-semibold">{item.title}</div>{/if}
									{#if item.subtitle}<div class="opacity-60">{item.subtitle}</div>{/if}
									{#if item.description}<p class="opacity-80">{item.description}</p>{/if}
								</div>
							{/each}
						{/if}
					</div>
				{/each}
			</div>
		{/if}
	</div>
</div>
