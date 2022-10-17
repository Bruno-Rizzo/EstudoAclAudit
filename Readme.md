
## Instalar o Pacote Breeze

- sail composer require laravel/breeze --dev
- sail artisan breeze:install

## Criar a classe Role com Migration e Controller Resouce

- sail artisan make:model Role -mrc

[Model]

    protected $fillable = ['name'];

[Migration]

    Schema::create('roles', function (Blueprint $table) {
                $table->id();
                $table->string('name');
                $table->timestamps();
    });

## Criar a classe Permission com Migration

- sail artisan make:model Permission -m

[Model]

    protected $fillable = ['name'];

[Migration]

    Schema::create('permissions', function (Blueprint $table) {
                $table->id();
                $table->string('name');
                $table->timestamps();
    });

## Criar a Migration permission_role

- sail artisan make:migration create_permission_role_table

[Migration]

    Schema::create('permission_role', function (Blueprint $table) {
                $table->foreignId('permission_id')->constrained('permissions')->onDelete('cascade');
                $table->foreignId('role_id')->constrained('roles')->onDelete('cascade');
    });

## Modificar a Migration usuarios

[Model]

    protected $fillable = [
        'name',
        'email',
        'password',
        'role_id'
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];
    
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

[Migration]

    Schema::create('users', function (Blueprint $table) {
                $table->id();
                $table->foreignId('role_id')->constrained('roles')->onDelete('cascade');
                $table->string('name');
                $table->string('email')->unique();
                $table->timestamp('email_verified_at')->nullable();
                $table->string('password');
                $table->rememberToken();
                $table->timestamps();
    });

## Criar os Relacionamentos com os Modelos

[Role]

    public function permissions()
    {
        return $this->belongsToMany(Permission::class);
    }

    public function users()
    {
        return $this->hasMany(User::class);
    }

[Permission]

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }

[User]

    public function role()
    {
        return $this->belongsTo(Role::class);
    }

## Criar um método para no Model Role para verificar se uma Função possui determinada Permissão

    public function hasPermission($name):bool   
    {
        return $this->permissions()->where('name',$name)->exists();
    }

## Criar o Model Post com Migration e Controller Resource

- sail artisan make:model Post -mrc

[Model]

    protected $fillable = [
        'title',
        'description',
        'author'
    ];

[Migration]

    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description');
            $table->text('author');
            $table->timestamps();
        });
    }


## Criar o seeder para popular a tabela inicial

- sail artisan make:seeder AdminSeeder
- sail artisan make:seeder PostSeeder

[AdminSeeder]

    public function run()
    {
        User::create([
            'name'              => 'Bruno Rizzo',
            'email'             => 'bruno@email.com',
            'email_verified_at' => now(),
            'password'          => bcrypt('password'),
            'role_id'           => Role::create(['name' => 'Administrador'])->id
        ]);

        User::create([
            'name'              => 'Ananda Cristina',
            'email'             => 'ananda@email.com',
            'email_verified_at' => now(),
            'password'          => bcrypt('password'),
            'role_id'           => Role::create(['name' => 'Gerente'])->id
        ]);

        User::create([
            'name'              => 'Luíza Cristina',
            'email'             => 'luiza@email.com',
            'email_verified_at' => now(),
            'password'          => bcrypt('password'),
            'role_id'           => Role::create(['name' => 'Analista'])->id
        ]);

        Permission::create(['name' => 'post-criar']);
        Permission::create(['name' => 'post-visualizar']);
        Permission::create(['name' => 'post-editar']);
        Permission::create(['name' => 'post-excluir']);

        DB::table('permission_role')->insert([
            ['permission_id' => '1' , 'role_id' => '1'],
            ['permission_id' => '2' , 'role_id' => '1'],
            ['permission_id' => '3' , 'role_id' => '1'],
            ['permission_id' => '4' , 'role_id' => '1'],
        ]);

    }

[PostSeeder]

    public function run()
    {
        Post::create([
            'title'       => 'Primeiro Post',
            'description' => 'Este é o primeiro post de teste',
            'author'      => 'Bruno Rizzo',
        ]);

        Post::create([
            'title'       => 'Segundo Post',
            'description' => 'Este é o segundo post de teste',
            'author'      => 'Luíza Cristina',
        ]);

        Post::create([
            'title'       => 'Terceiro Post',
            'description' => 'Este é o terceiro post de teste',
            'author'      => 'Luíza Cristina',
        ]);

        Post::create([
            'title'       => 'Quarto Post',
            'description' => 'Este é o quarto post de teste',
            'author'      => 'Ananda Cristina',
        ]);
    }

## Incluir os Seeders no DatabaseSeeder

    public function run()
    {
        $this->call([
            AdminSeeder::class,
            PostSeeder::class
        ]);
    }

## Rodar as Migratios Criando e Populando as Tabelas

- sail artisan migrate:fresh --seed

## Criar o Controller de usuários

- sail artisan make:controller UserController -r

## Criar as Rotas em routes/web.php

    Route::get('/', function () {
        return view('welcome');
    });

    Route::middleware('auth')->group(function(){

    Route::get('/dashboard', function () {
        return view('dashboard');
    })->name('dashboard');

    Route::middleware('admin')->group(function(){

        Route::resource('/users', UserController::class);
        Route::resource('/roles', RoleController::class);

    });

    Route::resource('/posts', PostController::class);

    });

## Criar a Página Master (layouts/app.blade.php) utilizando CDNs do Bootstrap, Toastr e Fontawesome


    <!doctype html>
    <html lang="pt-br">

    <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Estudo</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/css/bootstrap.min.css" rel="stylesheet"
          integrity="sha384-iYQeCzEYFbKjA/T2uDLTpkwGzCiq6soy8tYaI1GyVh/UjpbCx/TYkiZhlZB6+fzT" crossorigin="anonymous">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/toastr.js/latest/toastr.min.css">
    </head>

    <body style="background-color: #CCC">

    <nav class="navbar navbar-expand-lg bg-light">

        <div class="container-fluid">

            <a class="navbar-brand" href="#">Estudo</a>

            <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false"
                aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="navbarSupportedContent">

                <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                    <li class="nav-item">
                        <a class="nav-link active" aria-current="page" href="{{ route('dashboard') }}">Dashboard</a>
                    </li>
                        <li class="nav-item">
                            <a class="nav-link" aria-current="page" href="{{ route('posts.index') }}">Posts</a>
                        </li>
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"
                                aria-expanded="false">
                                Configurações
                            </a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="{{ route('users.index') }}">Usuários</a></li>
                                <li><a class="dropdown-item" href="{{ route('roles.index') }}">Funções</a></li>
                            </ul>
                        </li>
                </ul>

                <form class="d-flex" role="search" action="{{ route('logout') }}" method="post">
                    @csrf
                    <button class="btn btn-sm btn-secondary" type="submit">Logout</button>
                </form>

            </div>
        </div>
    </nav>

    <div class="container">
        @yield('content')
    </div>

    <script src="https://code.jquery.com/jquery-3.6.1.min.js" integrity="sha256-o88AwQnZB+VDvE9tvIXrMQaPlFFSUTR+nldQm1LuPXQ="       crossorigin="anonymous">
    </script>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/js/bootstrap.bundle.min.js"
            integrity="sha384-u1OknCvxWvY5kfmNBILK2hRnQC3Pr17a+RTT6rIHI7NnikvbZlHgTPOOmMi466C8" crossorigin="anonymous">
    </script>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/toastr.js/latest/toastr.min.js"></script>
    
    <script>
            @if(Session::has('success'))
            toastr.options =
            {
                "closeButton" : true,
                "progressBar" : true
            }
                    toastr.success("{{ session('success') }}");
            @endif

            @if(Session::has('error'))
            toastr.options =
            {
                "closeButton" : true,
                "progressBar" : true
            }
                    toastr.error("{{ session('error') }}");
            @endif

            @if(Session::has('info'))
            toastr.options =
            {
                "closeButton" : true,
                "progressBar" : true
            }
                    toastr.info("{{ session('info') }}");
            @endif

            @if(Session::has('warning'))
            toastr.options =
            {
                "closeButton" : true,
                "progressBar" : true
            }
                    toastr.warning("{{ session('warning') }}");
            @endif
    </script>  

    </body>

    </html>

## Criação dos sistemas de controle Middleware, Policy e Gates

- sail artisan make:middleware EnsureIsAdmin

        class EnsureIsAdmin
        {
        
            public function handle(Request $request, Closure $next)
            {
                if (Auth::user()->role_id != 1){
                    abort(403);
                }
                return $next($request);
            }
        }

Publicar en Kernel.php

    protected $routeMiddleware = [
        'admin' => \App\Http\Middleware\EnsureIsAdmin::class,
    ];

Em Providers/AuthServiceProvider.php

    public function boot()
    {
        $this->registerPolicies();

        Gate::define('admin', function(){
            return Auth::user()->role_id ==1 ? true : false;
        });
    }

- sail artisan make:policy PostPolicy --model=Post

        class PostPolicy
        {
            use HandlesAuthorization;

            public function view()
            {
                return Auth::user()->role->hasPermission('post-visualizar');
            }


            public function create()
            {
                return Auth::user()->role->hasPermission('post-criar');
            }


            public function update()
            {
                return Auth::user()->role->hasPermission('post-editar');
            }


            public function delete()
            {
                return Auth::user()->role->hasPermission('post-excluir');
            }
        }


## Criar as Páginas com CRUD (Exemplo: POST)

[PostController]

    class PostController extends Controller
    {
        
        public function index()
        {
            $this->authorize('view',Post::class);
            $posts = Post::OrderByDesc('id')->latest()->take(5)->get();
            return view('admin.posts.index',compact('posts'));
        }

        public function search(Request $request)
        {
            if(!empty($request->title)){
                $posts = Post::where('title','like',"%$request->title%")->get();
            }elseif(!empty($request->author)){
                $posts = Post::where('author','like',"%$request->author%")->get();
            }else{
                $posts = Post::OrderByDesc('id')->latest()->take(5)->get();
            }
            return view('admin.posts.index',compact('posts'));
        }
        
        public function create()
        {
            $this->authorize('create',Post::class);
            return view('admin.posts.create');
        }

        
        public function store(PostStoreRequest $request)
        {
            $posts = $request->validated();
            $posts['author'] = Auth::user()->name;
            Post::create($posts);

            return to_route('posts.index')->with('info','Post cadastrado com sucesso');
        }

        
        public function show(Post $post)
        {
            //
        }

    
        public function edit(Post $post)
        {
            $this->authorize('update',Post::class);
            return view('admin.posts.edit',compact('post'));
        }

    
        public function update(PostUpdateRequest $request, Post $post)
        {
            $posts = $request->validated();
            $posts['author'] = Auth::user()->name;
            $post->update($posts);
            
            return to_route('posts.index')->with('info','Post editado com sucesso');;
        }

    
        public function destroy(Post $post)
        {
            $this->authorize('delete',Post::class);
            $post->delete();
            return to_route('posts.index')->with('error','Post excluído com sucesso');
        }

    }

[Index-Post]

    @extends('layouts.app')

    @section('content')

    <div class="card mt-3">

        <div class="card-header">
            POSTS - Lista de Posts
        </div>

        <div class="card-body">

        @can('create',App\Models\Post::class)
        <a href="{{route('posts.create')}}" class="btn btn-sm btn-secondary">+ Novo Post</a>
        @endcan

        <form action="{{route('posts.search')}}" method="post">
        @csrf

        <div class="row mt-3">
            <div class="col-md-5">
                <input type="text" class="form-control" placeholder="Título" name="title">
            </div>
            <div class="col-md-5">
                <input type="text" class="form-control" placeholder="Autor" name="author">
            </div>
            <div class="col-md-2">
                <button type="submit" class="btn btn-sm btn-secondary">
                    <i class="fa fa-search m-1"></i> Pesquisar
                </button>
            </div>
        </div>

        </form>

    @if (Route::is('posts.search'))
        <p class="mt-2">Resultado da Pesquisa</p>
    @else
        <p class="mt-2">Lista dos últimos 5 Posts</p>
    @endif

    <table class="table table-sm table-bordered mt-3">
        <thead>
            <tr>
                <th class="text-center">#</th>
                <th>Título</th>
                <th>Autor</th>
                <th class="text-center">Data Criação</th>
                @can('update',App\Models\Post::class)
                <th class="text-center">Editar</th>
                @endcan
                @can('delete',App\Models\Post::class)
                <th class="text-center">Excluir</th>
                @endcan
            </tr>
        </thead>
        <tbody>
             @foreach ($posts as $post)
            <tr>
                <td class="text-center">{{$post->id}}</td>
                <td>{{$post->title}}</td>
                <td>{{$post->author}}</td>
                <td class="text-center">{{$post->created_at->format('d/m/Y')}}</td>
                @can('update',App\Models\Post::class)
                <td class="text-center">
                    <a href="{{route('posts.edit',$post->id)}}" class="btn btn-sm btn-success">
                        <i class="fa fa-edit"></i>
                    </a>
                </td>
                @endcan
                @can('delete',App\Models\Post::class)
                <td class="text-center">
                    <form action="{{route('posts.destroy',$post->id)}}" method="post">
                        @csrf
                        @method('delete')
                        <button type="submit" class="btn btn-sm btn-danger" onclick="return confirm('Deseja excluir este Post?')">
                            <i class="fa fa-trash"></i>
                        </button>
                    </form>
                </td>
                @endcan
            </tr>
            @endforeach
        </tbody>
    </table>

    </div>

    </div>

    @endsection

## Criar as Páginas com CRUD (Exemplo: Role)

[RoleController]

    class RoleController extends Controller
    {
    
    public function index()
    {
        $roles = Role::all();
        return view('admin.roles.index',compact('roles'));
    }

    
    public function create()
    {
        return view('admin.roles.create');
    }

    
    public function store(RoleFormRequest $request)
    {
        Role::create($request->validated());
        return to_route('roles.index')->with('message','Função cadastrada com sucesso!');
    }

    
    public function show(Role $role)
    {
        //
    }

    
    public function edit(Role $role)
    {
        $permissions = Permission::all();
        return view('admin.roles.edit',compact('role','permissions'));
    }

    
    public function update(RoleFormRequest $request, Role $role)
    {
        $role->update($request->validated());
        return to_route('roles.index')->with('info','Função editada com sucesso!');
    }

    
    public function destroy(Role $role)
    {
        $role->delete();
        return to_route('roles.index')->with('error','Função excluída com sucesso!');
    }

    public function assignPermissions(Request $request , Role $role)
    {
        $role->permissions()->sync($request->permissions);
        return to_route('roles.index')->with('info','Permissão associada com sucesso!');
    }

    }

[Role-Edit]

    @extends('layouts.app')

    @section('content')

    <div class="card mt-3">

        <div class="card-header">
            FUNÇÕES - Cadastrar Função
        </div>

        <form action="{{ route('roles.update', $role->id) }}" method="post">
            @csrf
            @method('put')

        <div class="card-body">

        <div class="row mb-3">
            <label class="col-sm-1 col-form-label">Nome</label>
            <div class="col-sm-6">
                <input type="text" class="form-control" name="name" value="{{ $role->name }}">
                @error('name') <span class="text-danger">{{$message}}</span> @enderror
            </div>
        </div>

    </div>

    <div class="card-footer">
        <button type="submit" class="btn btn-sm btn-success">Editar</button>
    </div>

        </form>

    </div>

    <div class="card mt-3">

        <div class="card-header">
            FUNÇÕES - Associar Permissões
        </div>

        <form action="{{ route('roles.permissions', $role->id) }}" method="post">
        @csrf
        <div class="card-body ms-2">

            <div class="row">
            @foreach ($permissions as $permission)
            <div class="form-check form-switch col-sm-3">
                    <input class="form-check-input" type="checkbox" role="switch" id="{{ $permission->id }}"
                        name="permissions[]" value="{{ $permission->id }}" @checked($role->hasPermission($permission->name))>
                    <label class="form-check-label" for="{{ $permission->id }}">
                        {{ $permission->name }}
                    </label>
            </div>
            @endforeach
        </div>

    </div>

    <div class="card-footer">
        <button type="submit" class="btn btn-sm btn-success">Associar</button>
        <a href="{{ route('roles.index') }}" class="btn btn-sm btn-secondary">Voltar</a>
    </div>

    </form>
    
    </div>
    @endsection

## Utilização da Biblioteca Laravel Auditing

- sail composer require owen-it/laravel-auditing

Em config/app.php

    'providers' => [

    /*
    * Package Service Providers...
    */

    OwenIt\Auditing\AuditingServiceProvider::class,

    ]

- sail artisan vendor:publish --provider "OwenIt\Auditing\AuditingServiceProvider" --tag="config"

- sail artisan vendor:publish --provider "OwenIt\Auditing\AuditingServiceProvider" --tag="migrations"

- sail artisan migrate

Para monitorar uma Classe Específica

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;
    use OwenIt\Auditing\Contracts\Auditable;  [**Importar_Esta_Classe**]

    class Post extends Model implements Auditable
    {
        use HasFactory;
        use \OwenIt\Auditing\Auditable; [**Importar_Esta_Classe**]

        protected $fillable = [
            'title',
            'description',
            'author'
        ];
    }

** Criar um Controller par Audits

    namespace App\Http\Controllers;

    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\DB;

    class AuditController extends Controller
    {
        public function index()
        {
            return view('admin.audits.index');
        }

        public function search(Request $request)
        {
            $audits = DB::table('audits')
                        ->whereDate('created_at','>=',$request->date_initial)
                        ->whereDate('created_at','<=',$request->date_final)
                        ->get();
            return view('admin.audits.index',compact('audits'));
        }

        public function show($id)
        {
            $audit = DB::table('audits')
                    ->where('id',$id)
                    ->get();
            
            if($audit->isNotEmpty()){
                $user = User::where('id',$audit[0]->user_id)->get();
                return view('admin.audits.show',compact('audit','user'));
            }
        }
    }

[Audit_Index]

    @extends('layouts.app')

    @section('content')

    <div class="card mt-3">

        <div class="card-header">
            Auditoria - Pesquisar Evento
        </div>

        <form action="{{ route('audits.search') }}" method="post">
        @csrf

        <div class="card-body">

            <div class="row">

                <div class="col-sm-5">
                    <input type="date" class="form-control" name="date_initial">
                </div>

                <div class="col-sm-5">
                    <input type="date" class="form-control" name="date_final">
                </div>

                <div class="col-sm-2">
                    <button type="submit" class="btn btn-sm btn-secondary">
                        <i class="fa fa-search m-1"></i> Pesquisar
                    </button>
                </div>

            </div>

        </div>

        </form>

    </div>

    @isset($audits)
    <div class="card mt-3">

        <div class="card-header">
            Auditoria - Resultado da Pesquisa
        </div>

        <div class="card-body">

            <table class="table table-sm table-bordered mt-3">
                <thead>
                    <tr>
                        <th class="text-center">#</th>
                        <th>Classe</th>
                        <th class="text-center">Evento</th>
                        <th>Data do Evento</th>
                        <th class="text-center">Visualizar</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach ($audits as $audit)
                    <tr>
                        <td class="text-center">{{ $audit->id }}</td>
                        <td>{{ $audit->auditable_type }}</td>
                        <td class="text-center">{{ $audit->event }}</td>
                        <td>{{ $audit->created_at }}</td>
                        <td class="text-center">
                            <a href="{{ route('audits.show', $audit->id) }}" target="_blank"
                                class="btn btn-sm btn-success">
                                <i class="fa fa-search"></i>
                            </a>
                        </td>
                    </tr>
                    @endforeach
                </tbody>
            </table>
        </div>
     </div>
    @endisset
    @endsection

[Audit-Show]

    @extends('layouts.app')

    @section('content')

    <div class="card mt-3">

        <div class="card-header">
            Auditoria - Visualizar Evento
        </div>

        <div class="card-body">

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Classe Auditada:</strong>
                </label>
                <div class="col-sm-4">
                    <input type="text" readonly class="form-control-plaintext" value="{{$audit[0]->auditable_type}}">
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Usuário Responsável:</strong>
                </label>
                <div class="col-sm-4">
                    <input type="text" readonly class="form-control-plaintext" value="{{$user[0]->name}}">
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Evento:</strong>
                </label>
                <div class="col-sm-4">
                    <input type="text" readonly class="form-control-plaintext" 
                    @if( $audit[0]->event == 'created' )
                        value="{{'Cadastro'}}"
                    @elseif($audit[0]->event == 'updated'){
                        value="{{'Edição'}}"
                    @else
                        value="{{'Exclusão'}}"
                    @endif
                  >
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Id Auditado:</strong>
                </label>
                <div class="col-sm-2">
                    <input type="text" readonly class="form-control-plaintext" value="{{$audit[0]->auditable_id}}">
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Valores Anteriores:</strong>
                </label>
                <div class="col-sm-7">
                    <textarea rows="3" readonly class="form-control-plaintext">{{$audit[0]->old_values}}</textarea>
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Novos Valores:</strong>
                </label>
                <div class="col-sm-7">
                    <textarea rows="3" readonly class="form-control-plaintext">{{$audit[0]->new_values}}</textarea>
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Url:</strong>
                </label>
                <div class="col-sm-7">
                    <input type="text" readonly class="form-control-plaintext" value="{{$audit[0]->url}}">
                </div>
            </div>

            <div class="row mb-3">
                <label class="col-sm-3 col-form-label">
                    <strong>Data:</strong>
                </label>
                <div class="col-sm-4">
                    <input type="text" readonly class="form-control-plaintext" value="{{$audit[0]->created_at}}">
                </div>
            </div>

        </div>
    </div>
    
    @endsection
