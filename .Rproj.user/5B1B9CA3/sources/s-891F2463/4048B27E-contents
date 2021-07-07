#https://www.census.gov/data/tables/time-series/demo/income-poverty/cps-pinc/pinc-03.html

#https://www.census.gov/programs-surveys/cps/data.html
#https://www.census.gov/programs-surveys/cps/data/tables.html
#https://www.census.gov/data/tables/time-series/demo/income-poverty/cps-pinc.html
##https://www.census.gov/data/tables/time-series/demo/income-poverty/cps-pinc/pinc-03.html






library(tidyverse)
library(readxl)
library(httr)
library(purrr)
library(RColorBrewer)
library(kableExtra)
library(scales)

#1 = Less than 9th grade
#2 = 9th-12th nongrad
#3 = GED
#4 = Some college
#5 = Assoc degree
#6 = Bachelors degree
#7 = Masters degree
#8 = Professional degree
#9 = Doctorate degree

DegreeLvl_Cont <- c(1, 2, 3, 4, 5, 6, 7, 8, 9)
DegreeLvl <- c('Less than 9th grade', '9th-12th nongrad', 'GED', 'Some college', 'Assoc degree', 'Bachelors degree', 'Masters degree', 'Professional degree', 'Doctorate degree')
DegreeData <- data.frame(DegreeLvl_Cont, DegreeLvl)

ProcessFile <- function(recURL) {
  GET(recURL, write_disk(tf <- tempfile(fileext = ".xlsx"))) #Get file and save to temp file
  df <- read_excel(tf, 1L) #Read Excel file
  RaceVal <- as.character(df[9, 1]) #Get text of Race value
  RaceVal <- gsub(' Alone', '', RaceVal) #Remove text
  RaceVal <- gsub(' \\(any race\\)', '', RaceVal) #Remove text
  df <- df %>% filter(row_number() %in% 16:56)  #Retrieve rows 16 - 56
  df$Income <- as.integer(seq(2500, length.out=nrow(df), by=2500)) #Create column 'Income' with incrementer for income to replace column 'A'
  df <- df[ -c(1,2,8) ] #Remove column 'A'
  df <- df[c(10,1,2,3,4,5,6,7,8,9)] #Reorder columns
  df <- df %>% pivot_longer(!Income, names_to = "Edu_Attainment_Lvl", values_to = "KCount")  #Pivot the data longer
  df$Edu_Attainment_Lvl <- gsub('...', '', df$Edu_Attainment_Lvl) #Remove the "..." auto added text
  df <- df %>% mutate_if(is.character, as.integer) #Change character columns to integer columns
  df <- df %>% mutate(Edu_Attainment_Lvl = ifelse(Edu_Attainment_Lvl < 8, Edu_Attainment_Lvl - 2, Edu_Attainment_Lvl - 3)) #Play with column header value to make continuous variable
  df['Race'] = RaceVal
  return(df)
}

# Files from here: https://www.census.gov/data/tables/time-series/demo/income-poverty/cps-pinc/pinc-03.html
vecURLs <- c("https://www2.census.gov/programs-surveys/cps/tables/pinc-03/2020/pinc03_1_1_2_1.xlsx",  # All Races
             "https://www2.census.gov/programs-surveys/cps/tables/pinc-03/2020/pinc03_1_1_2_3.xlsx",  # White alone
             "https://www2.census.gov/programs-surveys/cps/tables/pinc-03/2020/pinc03_1_1_2_6.xlsx",  # Black alone
             "https://www2.census.gov/programs-surveys/cps/tables/pinc-03/2020/pinc03_1_1_2_8.xlsx",  # Asian alone 
             "https://www2.census.gov/programs-surveys/cps/tables/pinc-03/2020/pinc03_1_1_2_9.xlsx") # Hispanic

Inc_By_EducLvl_Race <- map_dfr(vecURLs, ProcessFile) #Calls the function ProcessFile 
Inc_By_EducLvl_Race_Filt <- Inc_By_EducLvl_Race %>% filter(Race != "All Races") 
Inc_By_EducLvl_Race_Filt <- as.data.frame(Inc_By_EducLvl_Race_Filt) 

IER <- data.frame(Income=integer(),
                  Edu_Attainment_Lvl=integer(), 
                  Race=character(),
                  stringsAsFactors=FALSE) 

for (row in 1:nrow(Inc_By_EducLvl_Race_Filt)) {
  aIncome             <- Inc_By_EducLvl_Race_Filt[row, "Income"]
  aEdu_Attainment_Lvl <- Inc_By_EducLvl_Race_Filt[row, "Edu_Attainment_Lvl"]
  aKCount             <- Inc_By_EducLvl_Race_Filt[row, "KCount"]
  aRace               <- Inc_By_EducLvl_Race_Filt[row, "Race"]
  Income <- floor(runif(aKCount, min = aIncome - 2500, max = aIncome))
  nLen <- length(Income)
  Edu_Attainment_Lvl <- rep(c(aEdu_Attainment_Lvl), each = nLen)
  Race <- rep(c(aRace), each = nLen)
  aDF <- data.frame(Income, Edu_Attainment_Lvl, Race, stringsAsFactors=FALSE)
  IER <- rbind(IER, aDF)
} 

IER_wTxt <- inner_join(IER, DegreeData, by = c("Edu_Attainment_Lvl" = "DegreeLvl_Cont"))
IER_wTxt$DegreeLvl <- factor(IER_wTxt$DegreeLvl, levels=c('Less than 9th grade', '9th-12th nongrad', 'GED', 'Some college', 'Assoc degree', 'Bachelors degree', 'Masters degree', 'Professional degree', 'Doctorate degree'))
IER_wTxt$Edu_Attainment_Lvl <- as.factor(IER_wTxt$Edu_Attainment_Lvl)
IER_wTxt$Race <- factor(IER_wTxt$Race, levels=c('White', 'Asian', 'Black', 'Hispanic'))
 
ggplot(IER_wTxt, aes(x=DegreeLvl, y=Income, fill=Race)) + 
  geom_violin() +  
  scale_fill_brewer(palette="Set1") + 
  labs(y = "Yearly Income", x = "Degree Level") +
  theme(axis.text.x = element_text(face = "bold", size = 12, angle = 45, vjust=0.8),
        axis.text.y = element_text(face = "bold", size = 12),
        axis.title.x = element_text(face = "bold", size = 14),
        axis.title.y = element_text(face = "bold", size = 14),
        legend.title = element_text(face = "bold", size = 14),
        legend.text = element_text(face = "bold", size = 12)) + 
  scale_y_continuous(labels=scales::dollar_format()) + 
  facet_grid(vars(Race), scales = "free") + 
  theme(strip.text.x = element_text(face = "bold", size = 13),
        strip.text = element_text(face = "bold", size = 13))

 
IER_wTxt_GB <- IER_wTxt %>%
  group_by(DegreeLvl, Race) %>% 
  summarise(median_Inc = round((median(Income)), digits = 0))
IER_wTxt_GB$median_Inc = formatC(IER_wTxt_GB$median_Inc, format="d", big.mark = ",")
IER_wTxt_GB_PW <- IER_wTxt_GB %>% 
  pivot_wider(names_from = Race, values_from = median_Inc)
 
IER_wTxt_GB_PW %>%
  kbl(align = c("l","r","r","r","r")) %>%
  kable_styling()




# Extend the regression lines beyond the domain of the data
ggplot(IER_wTxt, aes(x=DegreeLvl, y=Income, color=factor(Race))) + 
  #geom_point(aes(size = KCount)) + 
  geom_point(aes()) + 
  facet_grid(vars(Race), scales = "free")
  #geom_point(shape=1) 
  #scale_colour_hue(l=50) + # Use a slightly darker palette than normal
  #geom_smooth(method=lm,   # Add linear regression lines
  #            se=FALSE,    # Don't add shaded confidence region
  #            fullrange=TRUE) # Extend regression lines


