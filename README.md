# final-304

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r, include=FALSE, echo=FALSE}
library(janitor)
library(tidyverse)
employment <- read_csv('1410022201_databaseLoadingData.csv')

```

```{r, include=FALSE, echo=FALSE}
	
covid <- read_csv('COVID19 cases (1).csv')
```

```{r Table 1, echo=FALSE}

#total covid cases by months
covid_clean <- covid %>%
  mutate(Episode_month = months(as.POSIXct(covid$`Episode Date`)))%>%
  filter(Classification=="CONFIRMED") 

month_count <- covid_clean %>%
  group_by(Episode_month)%>%
  summarise(count = n(), .groups = "keep")
month_count <-month_count %>%
  mutate(Episode_month = factor(Episode_month, levels = month.name))%>%
  arrange(Episode_month) %>%
  filter(Episode_month == "January"|Episode_month == "February"|Episode_month == "March"|Episode_month == "April"|Episode_month == "May"|Episode_month == "June"|Episode_month == "July"|Episode_month == "August")

month_count


```


```{r, echo=FALSE}

month_count %>% # data layer
  ggplot(aes(x = Episode_month, y = count, group = 1)) + # axes layer
  geom_line()+ 
  labs(  # annotations layer
    title = "COVID-19 Cases reported each month in Canada",
    y = "Number of cases", x = "Episode month ")+
  theme(axis.text.x = element_text(angle = 45, hjust=1))



```


```{r, echo=FALSE}
#employment
employment_employees <- employment %>%
  filter(UOM == "Persons") %>%
  select(REF_DATE,Estimate,UOM,GEO,VALUE) %>%
  filter(Estimate == "Employment for all employees"|Estimate == "Employment for hourly paid employees"|Estimate == "Employment for salaried employees")
  
employment_employees<-employment_employees[-c(9, 18,27),]
employment_employees
```

```{r echo=FALSE, warning=FALSE}

ggplot(employment_employees, aes(x=REF_DATE, y=VALUE)) + geom_line(aes(group =Estimate,colour=Estimate)) + theme(
  legend.title = element_text(size = 8),
  legend.text = element_text(size = 8),legend.position = c(.8, .55)
    )  + scale_colour_discrete(name = "Types of employees")+ 
  labs(  # annotations layer
    title = "Employment for all employees, hourly paid and salaried employees",
    y = "Number of employment", x = "Month ")


```




```{r, echo=FALSE}



all_employment <- employment_employees%>% 
  filter(Estimate == "Employment for all employees")

all_employment <- cbind(all_employment, month_count)
all_employment


```

```{r, echo=FALSE}
library(lme4)

lm1<-lm( all_employment$VALUE ~ all_employment$count )
summary(lm1)

confint(lm1)


```


```{r, echo=FALSE}
plot(all_employment$count,all_employment$VALUE,col = "blue",main = "COVID-19 cases versus employment for all employees",
abline(lm1),cex = 1.3,pch = 16,xlab = "Number of cases",ylab = "Number of employment")
```

```{r, echo=FALSE}

## Linear Regression Estimation for Survey Data
n=8
N=100
#install.packages("survey")
library(survey)
## Using the Survey Library
fpc.srs = rep(N, n)

all.design <- svydesign(id=~1, data=all_employment, fpc=fpc.srs)
  

svyratio(~all_employment$VALUE, ~all_employment$count, 
        design=all.design)


mysvylm <- svyglm(VALUE ~count, 
        design=all.design)

summary(mysvylm)
confint(mysvylm)
```


```{r, echo=FALSE}

hour_employment <- employment_employees %>% 
  filter(Estimate == "Employment for hourly paid employees")

hour_employment <- cbind(hour_employment, month_count)
hour_employment

```

```{r, echo=FALSE}
lm2<-lm( hour_employment$VALUE ~ hour_employment$count , data = hour_employment)
summary(lm2)
confint(lm2)

```

```{r, echo=FALSE}
plot(hour_employment$count,hour_employment$VALUE,col = "blue",main = "COVID-19 cases versus employment for hourly paid employees", 
     abline(lm2),cex = 1.3,pch = 16,xlab = "Number of cases",ylab = "Number of employment")
```

```{r, echo=FALSE}
salaried_employment <- employment_employees %>% 
  filter(Estimate == "Employment for salaried employees")

salaried_employment <- cbind(salaried_employment, month_count)
salaried_employment



```


```{r, echo=FALSE}
lm3<-lm( salaried_employment$VALUE ~ salaried_employment$count , data = salaried_employment)
summary(lm3)
confint(lm3)
```

```{r, echo=FALSE}
#employment of all employees 
#FEB- APRIL
a <-(13991504 - 16733202)/16733202*100
#APRIL-JULY
b <-(14855330 - 13991504)/13991504*100


#employment of hourly paid employees 
#FEB- APRIL
c<-(7463480 - 9772040)/9772040*100
#APRIL-JULY
d<-(8509873 - 7463480)/7463480*100

#employment of salaried employees 
#FEB- APRIL
e<-(5592299-5937018)/5937018*100
#APRIL-JULY
f<-(5614902 - 5592299)/5592299*100

increase_rate.data <- data.frame(
  date= c ("February- April","April-July"), 
   increase_rate = c(a,b,c,d,e,f),
  employment_type = c("all employees", "all employees","hourly paid employees","hourly paid employees", "salaried employees ","salaried employees ")
  )
increase_rate.data 



```



```{r, echo=FALSE}


ggplot(data=increase_rate.data, aes(x=employment_type, y=increase_rate
, fill=date)) +
geom_bar(stat="identity", position=position_dodge())+labs(  # annotations layer
    title = "Increase rate in employment for different type of employees",
    y = "Increase rate in employment", x = "Type of employees ")


```
