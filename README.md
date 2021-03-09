# Summing-Gas-Quantity
# code for reading massive gas production data sheet and summing overal gas and oil production
library(readxl)
library(writexl)
library(tidyr)
library(tidyverse)
library(lubridate)
library(zoo)

#find large data sheet on computer and read it into R
A <- file.choose()
MAIN_UNCONVENTIONAL_GAS_PROD_DF <- read_xlsx(A)

#use complete.cases function to remove all rows with NA's in
#GAS_QUANTITY column of our giant gas production data frame
MAIN_UNCONVENTIONAL_GAS_PROD_DF <- MAIN_UNCONVENTIONAL_GAS_PROD_DF[complete.cases(MAIN_UNCONVENTIONAL_GAS_PROD_DF$GAS_QUANTITY),]

#create data frame of unique WELL_PERMIT_NUM's for entire state of PA
#so we can fill it later with the total GAS_QUANTITY output over the 
#six years of data we have from our giant MAIN_UNCONVENTIONAL_GAS_PROD_D
TOTAL_OUTPUT_BY_WELL_PERMIT_NUM <- data.frame("WELL_PERMIT_NUM" = unique(MAIN_UNCONVENTIONAL_GAS_PROD_DF$WELL_PERMIT_NUM),"GAS_QUANTITY" = 0)

#here is the part that sucks
#running nested for loops to go through our massive data frame to sum all
#GAS_QUANTITY monthly outputs for each unique WELL_PERMIT_NUM takes forever
#but I have not been able to come up with a more effient way to sum all these
#totals
'for (x in 1:length(TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$WELL_PERMIT_NUM)) {
  for (y in 1:length(MAIN_UNCONVENTIONAL_GAS_PROD_DF$WELL_PERMIT_NUM)) {
    if(TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$WELL_PERMIT_NUM[x] == MAIN_UNCONVENTIONAL_GAS_PROD_DF$WELL_PERMIT_NUM[y])
      TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$GAS_QUANTITY[x] = 
      TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$GAS_QUANTITY[x] +
      MAIN_UNCONVENTIONAL_GAS_PROD_DF$GAS_QUANTITY[y]
  }
}'

#this previous attempt is total trash, FULL FORCE
#I have cracked the code in half
#summing totals based on WELL_PERMIT_NUMBER is a challenge no more
#Let's use the subset function inside the for loop to fill our GAS_QUANTITY
#totals in our TOTAL_OUTPUT_BY_WELL_PERMIT_NUM data frame with some help
#from the sum function

for (x in 1:length(TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$WELL_PERMIT_NUM)) {
  a <- subset(MAIN_UNCONVENTIONAL_GAS_PROD_DF,WELL_PERMIT_NUM == TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$WELL_PERMIT_NUM[x])
  TOTAL_OUTPUT_BY_WELL_PERMIT_NUM$GAS_QUANTITY[x] = sum(a$GAS_QUANTITY)
}

#so we have a data frame of all the GAS_QUANTITY summed output from 2015 - 2020
#for each unique WELL_PERMIT_NUM.  Lets arrange it from highest output to lowest 
#output and we are done.

TOTAL_OUTPUT_BY_WELL_PERMIT_NUM <- arrange(TOTAL_OUTPUT_BY_WELL_PERMIT_NUM,desc(GAS_QUANTITY))

