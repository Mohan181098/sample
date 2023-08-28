 
Select *
From `bamboo-truck-384707.coviddata.coviddeaths` 
Where continent is not null 
order by location,date;

-- Select Data that we are going to be starting with

Select location, date, total_cases, new_cases, total_deaths, population
From `bamboo-truck-384707.coviddata.coviddeaths` 
Where continent is not null 
order by location,date;

-- Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract covid in your country

Select location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From `bamboo-truck-384707.coviddata.coviddeaths`
Where location like '%India%'
and continent is not null 
order by location,date;

-- Total Cases vs Population
-- Shows what percentage of population infected with Covid

Select location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From `bamboo-truck-384707.coviddata.coviddeaths`
order by location,date;

-- Shows pecentage of people infected with covid in India
Select location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From `bamboo-truck-384707.coviddata.coviddeaths`
where location like  '%India%'
order by location,date;

-- Countries with Highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From `bamboo-truck-384707.coviddata.coviddeaths`
Group by Location, Population
order by PercentPopulationInfected desc;

--Infection rate in India

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From `bamboo-truck-384707.coviddata.coviddeaths`
where location like '%India%'
Group by Location, Population
order by PercentPopulationInfected desc;

-- Countries with Highest Death Count per Population

Select location, Max(total_deaths) as TotalDeathCount
From `bamboo-truck-384707.coviddata.coviddeaths`
Where continent is not null 
Group by Location
order by TotalDeathCount desc;


-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select location, MAX(Total_deaths) as TotalDeathCount
From `bamboo-truck-384707.coviddata.coviddeaths`
Where continent is null 
Group by location
order by TotalDeathCount desc;


-- GLOBAL NUMBERS

Select SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, SUM(new_deaths)/SUM(New_Cases)*100 as DeathPercentage
From `bamboo-truck-384707.coviddata.coviddeaths`
where continent is not null 
order by total_cases,total_deaths;

-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations)OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From `bamboo-truck-384707.coviddata.coviddeaths` dea
Join `bamboo-truck-384707.coviddata.covidvaccination`vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by location,date;

-- Using CTE to perform Calculation on Partition By in previous query
WITH PopvsVac AS (
    SELECT
        dea.continent,
        dea.location,
        dea.date,
        dea.population,
        vac.new_vaccinations,
        SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
    FROM `bamboo-truck-384707.coviddata.coviddeaths` dea
    JOIN `bamboo-truck-384707.coviddata.covidvaccination` vac
       ON dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated / Population) * 100
FROM PopvsVac;


-- Creating View to store data for later visualizations
CREATE OR REPLACE VIEW `bamboo-truck-384707.coviddata. PercentPopulationVaccinated` AS
SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
FROM
    `bamboo-truck-384707.coviddata.coviddeaths` dea
JOIN
    `bamboo-truck-384707.coviddata.covidvaccination` vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE
    dea.continent IS NOT NULL;
