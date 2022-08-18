# Covid-Data
An analysis of Covid 19 Data Worldwide from January 2020 to July 2022
select * from PortfolioProject..CovidDeath
where continent is not null
order by 1,2

select location, date, total_cases,new_cases,total_deaths, population 
from PortfolioProject..CovidDeath
where continent is not null
order by 1,2


--total casses vs total death
--Likely of dying if one is effected by covid
select location, date, total_cases,new_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from PortfolioProject..CovidDeath
where continent is not null
order by 1,2


select location, date, total_cases,new_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from PortfolioProject..CovidDeath
where location like 'Nigeria'
order by 1,2

-- Total Case vs Popuplation
-- Shows what percentage of the population got Covid
select location, date, population, total_cases, (total_cases/population)*100 as CasesPercentage 
from PortfolioProject..CovidDeath
where location = 'Nigeria'
order by 1,2

--Countries with the Higest infection rate compared to Population
select location, population, max(total_cases) as HighestInfectioncount, Max((total_cases/population))*100 as PercentofPopulationInfected
from PortfolioProject..CovidDeath
where continent is not null
group by Location, population
order by PercentofPopulationInfected Desc

--Countries with the highest Death count per population
select location, max(cast(total_deaths as int)) as TotalDeathcount
from PortfolioProject..CovidDeath
where continent is not null
group by Location
order by TotalDeathcount Desc

--Working with continent
select continent, max(cast(total_deaths as int)) as TotalDeathcount
from PortfolioProject..CovidDeath
where continent is not null
group by continent
order by TotalDeathcount Desc

--Global Numbers
select date, sum(new_cases) as total_Cases, sum(cast(new_deaths as int)) as Total_Death, sum(cast(new_deaths as int))/sum(new_cases)*100 as Death_Percentage
from PortfolioProject..CovidDeath
where continent is not null
group by date
order by 1,2


select sum(new_cases) as total_Cases, sum(cast(new_deaths as int)) as Total_Death, sum(cast(new_deaths as int))/sum(new_cases)*100 as Death_Percentage
from PortfolioProject..CovidDeath
where continent is not null
order by 1,2

--On Covid Vaccination and Covid death
select *
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date

--Total Population vs Total Vaccination
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date
where dea.continent is not null
order by 1,2,3

-- ROlling Count of People Vaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(cast(vac.new_vaccinations as bigint)) over (partition by dea.location order by dea.location, dea.date) as Rolling_People_vaccinated
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date
where dea.continent is not null
order by 2,3

--USE CTE
--Population VS Vaccination
with PopvsVac (continent, Location, Date, Population, New_vaccinations, Rolling_People_vaccinated)
as
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(cast(vac.new_vaccinations as bigint)) over (partition by dea.location order by dea.location, dea.date) as Rolling_People_vaccinated
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
select *, (Rolling_People_vaccinated/Population)*100
from PopvsVac

--Using Temp Table
drop table if exists #percentPopulationVaccinated
create table #percentPopulationVaccinated
(
continent nvarchar(255),
location nvarchar(255),
date datetime,
popuplation numeric,
new_vaccinated numeric,
Rolling_People_vaccinated numeric
)
insert into #percentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(cast(vac.new_vaccinations as bigint)) over (partition by dea.location order by dea.location, dea.date) as Rolling_People_vaccinated
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date
where dea.continent is not null
--order by 2,3
select * --, (Rolling_People_vaccinated/Population)*100
from #percentPopulationVaccinated

-- Creating Views to store data for visuals
CREATE view PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(cast(vac.new_vaccinations as bigint)) over (partition by dea.location order by dea.location, dea.date) as Rolling_People_vaccinated
from PortfolioProject..CovidDeath dea
join PortfolioProject..covidvaccination vac
on dea.location = vac.location and dea.date = vac.date
where dea.continent is not null
--order by 2,3

-- table 2 querry
select location, sum(cast(new_deaths as int)) as TotalDeathCount
from PortfolioProject..CovidDeath
where continent is null
and location not in ('world', 'European Union', 'international', 'Upper middle income', 'High income', 'Lower middle income', 'Low income')
group by location
order by TotalDeathCount desc

--Table 4
select location, population, date, max(total_cases) as HighestInfectioncount, Max((total_cases/population))*100 as PercentofPopulationInfected
from PortfolioProject..CovidDeath
--where continent is not null
group by Location, population, date
order by PercentofPopulationInfected Desc
