# PharmaGestion

Application Windows WPF hors connexion pour la gestion d’une pharmacie sur un poste unique.

## Architecture

- `src/PharmaGestion.App` : vues WPF/XAML, ressources graphiques et ViewModels MVVM.
- `src/PharmaGestion.Core` : entités, contexte EF Core, migrations et services métier transactionnels.
- `tests/PharmaGestion.Tests` : tests SQLite réels (pas une base simulée).
- `installer` : script Inno Setup et publication Windows.
- `docs` : guide utilisateur, sauvegarde/restauration et décisions métier.

Le choix est volontairement simple : deux projets de production, injection de dépendances native Microsoft et aucune infrastructure distante. Les prix et coûts sont des `decimal` en C# et sont convertis en entiers SQLite à l’échelle 100. Toute suppression relationnelle est restrictive afin de préserver l’historique.

## Prérequis

- Windows 10/11 x64.
- SDK .NET 10 pour développer. L’application publiée en mode autonome n’exige pas le runtime.
- Visual Studio 2026 avec la charge WPF, ou la CLI `dotnet`.

## Commandes exactes

```powershell
$env:DOTNET_CLI_HOME = Join-Path (Get-Location) '.dotnet-cli'
dotnet restore PharmaGestion.sln
dotnet build PharmaGestion.sln --configuration Release --no-restore --maxcpucount:1
dotnet test tests/PharmaGestion.Tests/PharmaGestion.Tests.csproj --configuration Release --no-build
dotnet run --project src/PharmaGestion.App/PharmaGestion.App.csproj
```

Mode démonstration isolé des données réelles :

```powershell
dotnet run --project src/PharmaGestion.App/PharmaGestion.App.csproj -- --demo
```

Au premier lancement, l’application demande de créer l’administrateur. Aucun mot de passe par défaut n’existe. Le mode démo utilise une autre base et demande lui aussi la création d’un compte.

## Base et migrations

La base réelle est `%LOCALAPPDATA%\PharmaGestion\pharmagestion.db`. Le mode WAL peut être activé sans rendre les sauvegardes incohérentes : le service emploie l’API SQLite Backup.

```powershell
dotnet tool restore
dotnet tool run dotnet-ef database update --project src/PharmaGestion.Core --startup-project src/PharmaGestion.Core
dotnet tool run dotnet-ef migrations add NomMigration --project src/PharmaGestion.Core --startup-project src/PharmaGestion.Core --output-dir Data/Migrations
```

Captures de validation visuelle :

```powershell
dotnet run --project src/PharmaGestion.App/PharmaGestion.App.csproj -- --capture-previews
# Capture isolée possible : tableau, produits ou vente
dotnet run --project src/PharmaGestion.App/PharmaGestion.App.csproj -- --capture-preview=produits
```

## Publication

```powershell
powershell -ExecutionPolicy Bypass -File installer/publier.ps1
```

Le résultat autonome se trouve dans `artifacts/publish/win-x64`. Ouvrir ensuite `installer/PharmaGestion.iss` avec Inno Setup 6 et compiler le script. Voir [DEPENDANCES.md](docs/DEPENDANCES.md) et [DECISIONS_METIER.md](docs/DECISIONS_METIER.md).

Les résultats réellement exécutés et les limites matérielles sont consignés dans [VALIDATION.md](docs/VALIDATION.md). Les trois aperçus WPF 1366×768 sont dans `docs/previews`.
