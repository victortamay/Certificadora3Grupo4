// Configuração do Supabase
const SUPABASE_URL = "https://wivoneicusyqxfjfjpiq.supabase.co";
const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Indpdm9uZWljdXN5cXhmamZqcGlxIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM3MDUzNDgsImV4cCI6MjA4OTI4MTM0OH0.CIU-jF1gsufCL503YJD2H-wHu6T81QXFh7AwQbD7R8U";
// A biblioteca carregada via CDN define a variável global 'supabase'
// Para evitar conflito ao reatribuir 'const supabase', usamos um nome diferente para o cliente
let db;
try {
    db = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
} catch (e) {
    console.error("Erro ao inicializar Supabase:", e);
}

// Estado Global
const state = {
    user: null,
    isAdmin: false,
    activities: [],
    loading: true
};

// --- Funções de Utilidade ---

function getEffectiveStatus(activity) {
    if (activity.status === 'cancelada') return 'cancelada';
    if (activity.status === 'encerrada' || new Date(activity.end_date) < new Date()) return 'encerrada';
    if (activity.available_slots <= 0) return 'lotada';
    return 'aberta';
}

function formatDate(dateStr) {
    const date = new Date(dateStr);
    return date.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit', year: 'numeric' });
}

function formatTime(dateStr) {
    const date = new Date(dateStr);
    return date.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' });
}

// --- Componentes ---

function renderNavbar() {
    const navContainer = document.getElementById('navbar-container');
    const user = state.user;
    const isAdmin = state.isAdmin;

    navContainer.innerHTML = `
        <nav class="border-b bg-card/80 backdrop-blur-sm sticky top-0 z-50">
            <div class="container mx-auto px-4 h-16 flex items-center justify-between">
                <a href="#/" class="flex items-center gap-2">
                    <div class="h-8 w-8 rounded-lg bg-primary flex items-center justify-center">
                        <span class="text-primary-foreground font-heading font-bold text-sm">MD</span>
                    </div>
                    <span class="font-heading font-semibold text-lg hidden sm:block">Meninas Digitais</span>
                </a>

                <div class="hidden md:flex items-center gap-6">
                    <a href="#/" class="nav-link text-sm font-medium text-muted-foreground hover:text-foreground transition-colors">Início</a>
                    <a href="#/atividades" class="nav-link text-sm font-medium text-muted-foreground hover:text-foreground transition-colors">Atividades</a>
                    ${user ? `<a href="#/minhas-inscricoes" class="nav-link text-sm font-medium text-muted-foreground hover:text-foreground transition-colors">Minhas Inscrições</a>` : ''}
                    ${isAdmin ? `<a href="#/admin" class="nav-link text-sm font-medium text-primary hover:text-primary/80 transition-colors">Painel Admin</a>` : ''}
                </div>

                <div class="hidden md:flex items-center gap-3">
                    ${user ? `
                        <a href="#/perfil" class="btn btn-ghost gap-2">
                            <i data-lucide="user" class="h-4 w-4"></i>
                            ${user.name?.split(' ')[0] || 'Perfil'}
                        </a>
                        <button id="logout-btn" class="btn btn-outline px-2">
                            <i data-lucide="log-out" class="h-4 w-4"></i>
                        </button>
                    ` : `
                        <a href="#/login" class="btn btn-ghost">Entrar</a>
                        <a href="#/cadastro" class="btn btn-primary">Cadastrar</a>
                    `}
                </div>

                <button id="mobile-menu-btn" class="md:hidden btn btn-ghost p-2">
                    <i data-lucide="menu" class="h-5 w-5"></i>
                </button>
            </div>
            <div id="mobile-menu" class="hidden md:hidden border-t bg-card p-4 space-y-3">
                <a href="#/" class="block text-sm font-medium py-2">Início</a>
                <a href="#/atividades" class="block text-sm font-medium py-2">Atividades</a>
                ${user ? `<a href="#/minhas-inscricoes" class="block text-sm font-medium py-2">Minhas Inscrições</a>` : ''}
                ${isAdmin ? `<a href="#/admin" class="block text-sm font-medium text-primary py-2">Painel Admin</a>` : ''}
                <div class="pt-2 border-t flex gap-2">
                    ${user ? `
                        <a href="#/perfil" class="btn btn-ghost btn-sm">Perfil</a>
                        <button onclick="db.auth.signOut()" class="btn btn-outline btn-sm">Sair</button>
                    ` : `
                        <a href="#/login" class="btn btn-ghost btn-sm">Entrar</a>
                        <a href="#/cadastro" class="btn btn-primary btn-sm">Cadastrar</a>
                    `}
                </div>
            </div>
        </nav>
    `;

    lucide.createIcons();

    const mobileBtn = document.getElementById('mobile-menu-btn');
    const mobileMenu = document.getElementById('mobile-menu');
    if (mobileBtn && mobileMenu) {
        mobileBtn.onclick = () => {
            mobileMenu.classList.toggle('hidden');
        };
    }

    const logoutBtn = document.getElementById('logout-btn');
    if (logoutBtn) {
        logoutBtn.onclick = async () => {
            await db.auth.signOut();
            window.location.hash = '#/';
        };
    }
}

function renderActivityCard(activity) {
    const status = getEffectiveStatus(activity);
    const isInactive = status === 'encerrada' || status === 'cancelada';
    
    const statusColors = {
        aberta: 'bg-emerald-500 text-white',
        lotada: 'bg-amber-500 text-white',
        encerrada: 'bg-slate-500 text-white',
        cancelada: 'bg-red-500 text-white'
    };

    return `
        <a href="#/atividades/${activity.id}" class="card overflow-hidden transition-all hover:shadow-md ${isInactive ? 'opacity-75 grayscale-[0.5]' : ''}">
            <div class="p-5">
                <div class="flex items-center justify-between mb-3">
                    <span class="badge bg-primary/10 text-primary border-none capitalize">${activity.type}</span>
                    <span class="badge ${statusColors[status]} border-none capitalize">${status}</span>
                </div>
                <h3 class="font-heading font-bold text-xl mb-2 line-clamp-2">${activity.title}</h3>
                <p class="text-muted-foreground text-sm line-clamp-2 mb-4">${activity.description}</p>
                <div class="space-y-2">
                    <div class="flex items-center gap-2 text-sm text-muted-foreground">
                        <i data-lucide="calendar" class="h-4 w-4"></i>
                        <span>${formatDate(activity.start_date)} às ${formatTime(activity.start_date)}</span>
                    </div>
                    <div class="flex items-center gap-2 text-sm text-muted-foreground">
                        <i data-lucide="map-pin" class="h-4 w-4"></i>
                        <span class="truncate">${activity.location}</span>
                    </div>
                </div>
            </div>
            <div class="px-5 py-3 border-t bg-muted/30 flex items-center justify-between">
                <span class="text-xs font-medium text-muted-foreground">
                    ${activity.available_slots === 0 ? 'Sem vagas' : `${activity.available_slots}/${activity.total_slots} vagas`}
                </span>
                <i data-lucide="arrow-right" class="h-4 w-4 text-primary"></i>
            </div>
        </a>
    `;
}

// --- Páginas ---

async function pageHome() {
    const { data: activities } = await db.from('activities').select('*').order('start_date', { ascending: true });
    const upcoming = (activities || []).filter(a => getEffectiveStatus(a) === 'aberta').slice(0, 3);

    return `
        <section class="relative overflow-hidden bg-gradient-to-br from-primary/5 via-secondary/5 to-accent py-20 lg:py-32">
            <div class="container mx-auto px-4">
                <div class="max-w-3xl">
                    <h1 class="font-heading text-4xl md:text-5xl lg:text-6xl font-bold tracking-tight mb-6">
                        Inspirando <span class="text-primary">meninas</span> a transformar o mundo com <span class="text-secondary">tecnologia</span>
                    </h1>
                    <p class="text-lg md:text-xl text-muted-foreground mb-8 max-w-2xl">
                        O projeto Meninas Digitais UTFPR-CP promove oficinas, minicursos e palestras para despertar o interesse de meninas e mulheres pela área de computação e tecnologia.
                    </p>
                    <div class="flex flex-wrap gap-4">
                        <a href="#/atividades" class="btn btn-primary btn-lg gap-2">
                            Ver atividades <i data-lucide="arrow-right" class="h-4 w-4"></i>
                        </a>
                        <a href="#/cadastro" class="btn btn-outline btn-lg">Cadastre-se</a>
                    </div>
                </div>
            </div>
        </section>

        <section class="py-16 border-b">
            <div class="container mx-auto px-4">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8 text-center">
                    <div>
                        <i data-lucide="book-open" class="h-8 w-8 text-primary mx-auto mb-3"></i>
                        <div class="font-heading text-3xl font-bold mb-1">${activities?.length || 0}</div>
                        <div class="text-sm text-muted-foreground">Oficinas, minicursos e palestras</div>
                    </div>
                    <div>
                        <i data-lucide="users" class="h-8 w-8 text-primary mx-auto mb-3"></i>
                        <div class="font-heading text-3xl font-bold mb-1">200+</div>
                        <div class="text-sm text-muted-foreground">Meninas impactadas pelo projeto</div>
                    </div>
                    <div>
                        <i data-lucide="sparkles" class="h-8 w-8 text-primary mx-auto mb-3"></i>
                        <div class="font-heading text-3xl font-bold mb-1">2020</div>
                        <div class="text-sm text-muted-foreground">Inspirando meninas na tecnologia</div>
                    </div>
                </div>
            </div>
        </section>

        <section class="py-16">
            <div class="container mx-auto px-4">
                <div class="flex items-center justify-between mb-8">
                    <div>
                        <h2 class="font-heading text-2xl md:text-3xl font-bold">Próximas Atividades</h2>
                        <p class="text-muted-foreground mt-1">Confira as atividades disponíveis para inscrição</p>
                    </div>
                    <a href="#/atividades" class="btn btn-outline gap-2 hidden sm:flex">
                        Ver todas <i data-lucide="arrow-right" class="h-4 w-4"></i>
                    </a>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    ${upcoming.map(renderActivityCard).join('')}
                </div>
            </div>
        </section>
    `;
}

async function pageActivities() {
    const { data: activities } = await db.from('activities').select('*').order('start_date', { ascending: true });
    
    return `
        <div class="container mx-auto px-4 py-12">
            <div class="mb-12">
                <h1 class="font-heading text-3xl md:text-4xl font-bold mb-4">Nossas Atividades</h1>
                <p class="text-muted-foreground max-w-2xl">
                    Participe de nossas atividades e mergulhe no mundo da tecnologia. Todas as atividades são gratuitas e voltadas para o público feminino.
                </p>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                ${(activities || []).map(renderActivityCard).join('')}
            </div>
        </div>
    `;
}

async function pageLogin() {
    return `
        <div class="container mx-auto px-4 py-20 flex justify-center">
            <div class="card w-full max-w-md p-8">
                <h1 class="font-heading text-2xl font-bold text-center mb-6">Entrar no Portal</h1>
                <form id="login-form" class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium mb-1">E-mail</label>
                        <input type="email" id="email" class="input-field" required placeholder="seu@email.com">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Senha</label>
                        <input type="password" id="password" class="input-field" required placeholder="••••••••">
                    </div>
                    <button type="submit" class="btn btn-primary w-full">Entrar</button>
                </form>
                <p class="text-center text-sm text-muted-foreground mt-6">
                    Não tem uma conta? <a href="#/cadastro" class="text-primary font-medium">Cadastre-se</a>
                </p>
            </div>
        </div>
    `;
}

async function pageRegister() {
    return `
        <div class="container mx-auto px-4 py-20 flex justify-center">
            <div class="card w-full max-w-md p-8">
                <h1 class="font-heading text-2xl font-bold text-center mb-6">Criar Conta</h1>
                <form id="register-form" class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium mb-1">Nome Completo</label>
                        <input type="text" id="name" class="input-field" required placeholder="Seu nome">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">E-mail</label>
                        <input type="email" id="email" class="input-field" required placeholder="seu@email.com">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Senha</label>
                        <input type="password" id="password" class="input-field" required minlength="6" placeholder="Mínimo 6 caracteres">
                    </div>
                    <button type="submit" class="btn btn-primary w-full">Criar Conta</button>
                </form>
                <p class="text-center text-sm text-muted-foreground mt-6">
                    Já tem uma conta? <a href="#/login" class="text-primary font-medium">Entrar</a>
                </p>
            </div>
        </div>
    `;
}

async function pageActivityDetail(id) {
    const { data: activity } = await db.from('activities').select('*').eq('id', id).single();
    if (!activity) return `<div class="p-20 text-center">Atividade não encontrada.</div>`;

    const status = getEffectiveStatus(activity);
    
    return `
        <div class="container mx-auto px-4 py-12">
            <a href="#/atividades" class="btn btn-ghost mb-8 gap-2">
                <i data-lucide="arrow-left" class="h-4 w-4"></i> Voltar
            </a>
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-12">
                <div class="lg:col-span-2">
                    <div class="flex items-center gap-3 mb-4">
                        <span class="badge bg-primary/10 text-primary capitalize">${activity.type}</span>
                        <span class="badge bg-slate-100 text-slate-700 capitalize">${status}</span>
                    </div>
                    <h1 class="font-heading text-4xl font-bold mb-6">${activity.title}</h1>
                    <div class="prose max-w-none">
                        <p class="text-lg text-muted-foreground leading-relaxed">${activity.description}</p>
                    </div>
                </div>
                <div>
                    <div class="card p-6 sticky top-24">
                        <h3 class="font-heading font-bold text-xl mb-4">Informações</h3>
                        <div class="space-y-4 mb-6">
                            <div class="flex items-start gap-3">
                                <i data-lucide="calendar" class="h-5 w-5 text-primary mt-0.5"></i>
                                <div>
                                    <div class="font-medium">Data</div>
                                    <div class="text-sm text-muted-foreground">${formatDate(activity.start_date)}</div>
                                </div>
                            </div>
                            <div class="flex items-start gap-3">
                                <i data-lucide="clock" class="h-5 w-5 text-primary mt-0.5"></i>
                                <div>
                                    <div class="font-medium">Horário</div>
                                    <div class="text-sm text-muted-foreground">${formatTime(activity.start_date)} às ${formatTime(activity.end_date)}</div>
                                </div>
                            </div>
                            <div class="flex items-start gap-3">
                                <i data-lucide="map-pin" class="h-5 w-5 text-primary mt-0.5"></i>
                                <div>
                                    <div class="font-medium">Local</div>
                                    <div class="text-sm text-muted-foreground">${activity.location}</div>
                                </div>
                            </div>
                            <div class="flex items-start gap-3">
                                <i data-lucide="users" class="h-5 w-5 text-primary mt-0.5"></i>
                                <div>
                                    <div class="font-medium">Vagas</div>
                                    <div class="text-sm text-muted-foreground">${activity.available_slots} disponíveis de ${activity.total_slots}</div>
                                </div>
                            </div>
                        </div>
                        <button id="register-activity-btn" class="btn btn-primary w-full btn-lg" ${status !== 'aberta' ? 'disabled' : ''}>
                            ${status === 'aberta' ? 'Inscrever-se' : 'Inscrições Indisponíveis'}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    `;
}

async function pageProfile() {
    if (!state.user) {
        window.location.hash = '#/login';
        return '';
    }
    return `
        <div class="container mx-auto px-4 py-8 max-w-2xl">
            <h1 class="font-heading text-3xl font-bold mb-8">Meu Perfil</h1>
            <div class="card p-8">
                <div class="flex items-center gap-4 mb-6">
                    <div class="h-16 w-16 rounded-full bg-primary/10 flex items-center justify-center">
                        <i data-lucide="user" class="h-8 w-8 text-primary"></i>
                    </div>
                    <div>
                        <h2 class="font-heading text-xl font-semibold">${state.user.name}</h2>
                        <p class="text-muted-foreground">${state.user.email}</p>
                    </div>
                </div>
                <div>
                    <p class="text-sm text-muted-foreground mb-1">Tipo de conta</p>
                    <span class="badge ${state.user.user_type === 'interno' ? 'bg-primary text-white' : 'bg-slate-200 text-slate-700'}">
                        ${state.user.user_type === 'interno' ? 'Integrante' : 'Externa'}
                    </span>
                </div>
            </div>
        </div>
    `;
}

async function pageMyRegistrations() {
    if (!state.user) {
        window.location.hash = '#/login';
        return '';
    }

    const { data: registrations } = await db
        .from('registrations')
        .select('*, activity:activities(*)')
        .eq('user_id', state.user.id)
        .neq('status', 'cancelada');

    return `
        <div class="container mx-auto px-4 py-8">
            <h1 class="font-heading text-3xl font-bold mb-2">Minhas Inscrições</h1>
            <p class="text-muted-foreground mb-8">Acompanhe suas atividades inscritas</p>
            <div class="space-y-4">
                ${(registrations || []).length === 0 ? `
                    <div class="text-center py-16">
                        <p class="text-muted-foreground text-lg mb-4">Você ainda não se inscreveu em nenhuma atividade.</p>
                        <a href="#/atividades" class="btn btn-primary">Explorar atividades</a>
                    </div>
                ` : registrations.map(reg => `
                    <div class="card p-6">
                        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                            <div class="flex-1">
                                <div class="flex flex-wrap gap-2 mb-2">
                                    <span class="badge bg-primary/10 text-primary capitalize">${reg.activity.type}</span>
                                    <span class="badge bg-slate-100 text-slate-700 capitalize">${getEffectiveStatus(reg.activity)}</span>
                                    <span class="badge bg-emerald-100 text-emerald-700 capitalize">${reg.status}</span>
                                </div>
                                <a href="#/atividades/${reg.activity.id}" class="font-heading font-semibold text-lg hover:text-primary transition-colors">
                                    ${reg.activity.title}
                                </a>
                                <div class="flex flex-wrap gap-4 mt-2 text-sm text-muted-foreground">
                                    <span class="flex items-center gap-1"><i data-lucide="calendar" class="h-3.5 w-3.5"></i> ${formatDate(reg.activity.start_date)}</span>
                                    <span class="flex items-center gap-1"><i data-lucide="map-pin" class="h-3.5 w-3.5"></i> ${reg.activity.location}</span>
                                </div>
                            </div>
                            ${reg.status === 'inscrita' ? `
                                <button onclick="cancelRegistration('${reg.id}')" class="btn btn-outline text-destructive hover:text-destructive gap-1">
                                    <i data-lucide="x" class="h-4 w-4"></i> Cancelar
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `).join('')}
            </div>
        </div>
    `;
}

async function pageAdmin() {
    if (!state.user || !state.isAdmin) {
        window.location.hash = '#/';
        return '';
    }

    const { data: activities } = await db.from('activities').select('*');
    const { data: regCounts } = await db.from('registrations').select('status').neq('status', 'cancelada');

    const totalActivities = activities?.length || 0;
    const totalRegistrations = regCounts?.length || 0;
    const activeRegistrations = regCounts?.filter(r => r.status === 'inscrita' || r.status === 'confirmada').length || 0;
    
    const totalSlots = activities?.reduce((s, a) => s + a.total_slots, 0) || 0;
    const availableSlots = activities?.reduce((s, a) => s + a.available_slots, 0) || 0;
    const occupancyRate = totalSlots > 0 ? Math.round((1 - availableSlots / totalSlots) * 100) : 0;

    return `
        <div class="container mx-auto px-4 py-8">
            <h1 class="font-heading text-3xl font-bold mb-2">Painel Admin</h1>
            <p class="text-muted-foreground mb-8">Visão geral do projeto Meninas Digitais</p>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
                <div class="card p-6">
                    <p class="text-sm text-muted-foreground">Atividades</p>
                    <p class="text-2xl font-heading font-bold mt-1">${totalActivities}</p>
                </div>
                <div class="card p-6">
                    <p class="text-sm text-muted-foreground">Inscrições</p>
                    <p class="text-2xl font-heading font-bold mt-1">${totalRegistrations}</p>
                </div>
                <div class="card p-6">
                    <p class="text-sm text-muted-foreground">Inscrições Ativas</p>
                    <p class="text-2xl font-heading font-bold mt-1">${activeRegistrations}</p>
                </div>
                <div class="card p-6">
                    <p class="text-sm text-muted-foreground">Taxa de Ocupação</p>
                    <p class="text-2xl font-heading font-bold mt-1">${occupancyRate}%</p>
                </div>
            </div>
            
            <div class="card p-6">
                <h3 class="font-heading font-bold text-xl mb-4">Gerenciamento</h3>
                <p class="text-muted-foreground mb-4">Para gerenciar atividades e inscrições detalhadamente, use o painel administrativo original ou implemente os módulos de CRUD aqui.</p>
                <div class="flex gap-4">
                    <button class="btn btn-primary" disabled>Nova Atividade</button>
                    <button class="btn btn-outline" disabled>Ver Todas Inscrições</button>
                </div>
            </div>
        </div>
    `;
}

// --- Roteador ---

const routes = {
    '/': pageHome,
    '/atividades': pageActivities,
    '/login': pageLogin,
    '/cadastro': pageRegister,
    '/atividades/:id': pageActivityDetail,
    '/perfil': pageProfile,
    '/minhas-inscricoes': pageMyRegistrations,
    '/admin': pageAdmin
};

async function router() {
    const hash = window.location.hash.slice(1) || '/';
    const mainContent = document.getElementById('main-content');
    
    // Loader simples
    mainContent.innerHTML = '<div class="flex justify-center py-20"><div class="animate-spin rounded-full h-8 w-8 border-b-2 border-primary"></div></div>';

    let pageFn = routes[hash];
    let params = null;

    // Lógica simples de params (ex: /atividades/:id)
    if (!pageFn) {
        for (const route in routes) {
            if (route.includes(':')) {
                const parts = route.split('/');
                const hashParts = hash.split('/');
                if (parts.length === hashParts.length && parts[1] === hashParts[1]) {
                    pageFn = routes[route];
                    params = hashParts[2];
                    break;
                }
            }
        }
    }

    if (pageFn) {
        mainContent.innerHTML = await pageFn(params);
        lucide.createIcons();
        setupPageListeners(hash);
    } else {
        mainContent.innerHTML = '<div class="p-20 text-center"><h1>404</h1><p>Página não encontrada.</p></div>';
    }
    
    renderNavbar();
}

async function cancelRegistration(registrationId) {
    if (!confirm('Tem certeza que deseja cancelar sua inscrição?')) return;
    
    try {
        const { data, error } = await db.rpc('cancel_registration', {
            p_registration_id: registrationId,
            p_user_id: state.user.id
        });

        if (error) throw error;
        
        if (data.success) {
            alert('Inscrição cancelada com sucesso!');
            router(); // Recarregar página
        } else {
            alert(data.message);
        }
    } catch (err) {
        alert('Erro ao cancelar: ' + err.message);
    }
}

function setupPageListeners(hash) {
    if (hash === '/login') {
        const form = document.getElementById('login-form');
        form.onsubmit = async (e) => {
            e.preventDefault();
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            const { error } = await db.auth.signInWithPassword({ email, password });
            if (error) alert(error.message);
            else window.location.hash = '#/';
        };
    } else if (hash === '/cadastro') {
        const form = document.getElementById('register-form');
        form.onsubmit = async (e) => {
            e.preventDefault();
            const name = document.getElementById('name').value;
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            const { error } = await db.auth.signUp({ 
                email, 
                password,
                options: { data: { name, user_type: 'externo' } }
            });
            if (error) alert(error.message);
            else alert('Conta criada! Verifique seu e-mail.');
        };
    } else if (hash.startsWith('/atividades/')) {
        const regBtn = document.getElementById('register-activity-btn');
        if (regBtn) {
            regBtn.onclick = async () => {
                if (!state.user) {
                    alert('Você precisa estar logada para se inscrever.');
                    window.location.hash = '#/login';
                    return;
                }

                const activityId = hash.split('/')[2];
                try {
                    const { data, error } = await db.rpc('register_for_activity', {
                        p_activity_id: activityId,
                        p_user_id: state.user.id
                    });

                    if (error) throw error;
                    
                    if (data.success) {
                        alert('Inscrição realizada com sucesso!');
                        window.location.hash = '#/minhas-inscricoes';
                    } else {
                        alert(data.message);
                    }
                } catch (err) {
                    alert('Erro na inscrição: ' + err.message);
                }
            };
        }
    }
}

// Inicialização
window.addEventListener('hashchange', router);
window.addEventListener('load', async () => {
    // Verificar sessão inicial
    const { data: { session } } = await db.auth.getSession();
    if (session) {
        state.user = { id: session.user.id, ...session.user.user_metadata };
        const { data: role } = await db.from('user_roles').select('role').eq('user_id', session.user.id).eq('role', 'admin').maybeSingle();
        state.isAdmin = !!role;
    }

    // Escutar mudanças de auth
    db.auth.onAuthStateChange((event, session) => {
        if (session) {
            state.user = { id: session.user.id, ...session.user.user_metadata };
        } else {
            state.user = null;
            state.isAdmin = false;
        }
        renderNavbar();
    });

    router();
});
