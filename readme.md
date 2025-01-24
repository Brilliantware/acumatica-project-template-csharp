# MyProject

MyProject description ... 

### Template usage
```bash
dotnet new acuprotem -o My.New.Project --AcumaticaInstancePath "c:\Program Files\Acumatica ERP\A242060014"
```
For a preconfigured '.env' file for './px' when publishing --online
```sh
dotnet new acuprotem -o My.New.Project --AcumaticaInstancePath "c:\Program Files\Acumatica ERP\A242060014" --AcumaticaBaseUrl http://localhost/A242060014 --AcumaticaUsername admin --AcumaticaPassword password --AcumaticaCompany Company
```

#### Notes:

<pre><i>Do not check in 'your' '.env'