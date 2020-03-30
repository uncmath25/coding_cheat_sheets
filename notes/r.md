# R

### Basics
* ` rm(list=ls()) ` clean environment
* ` require(LIBRARY_NAME) ` import library (outputs warning inside function)
* ` source(SCRIPT_PATH) ` source another R script
* ` data() ` check catalog of sample data sets, these sets can be immediately accessed by their names
* ` class(obj) ` gives data type

### Dataframes
* ` dim(df) ` shape of dataframe
* ` head(df) ` # head of dataframe
* ` names(df) ` column names
* ` ncol(df) ` column count
* ` nrow(df) ` row count
* ` df[1:3, 4:dim(df)[2]] ` sub-dataframe
* ` df[, -i] ` sub-dataframe with ith column removed
* ` df['col_name'] ` access columns as vector (also `df$col_name`)
* ` CO2[which(CO2$Treatment == 'chilled'), names(CO2) %in% c('conc', 'uptake')] ` logical subsetting
* ` table(df$col_name) ` frequency count for column name
* ` for (name in names(df)) {print(name); print(table(df[name]))} ` gives frequency count for all columns
* ` summary(df$col_name) ` gives nice percentile breakdown

### Matrices
* ` matrix(1:6, 2, 3) ` matrix(data, nrow, ncol)
* ` t(matrix) ` transpose
* ` rbind() ` build matrix row-wise from vectors
* ` cbind() ` build matrix column-wise from vectors
* ` data.frame(matrix(1:6, 2, 3)) ` create dataframe from matrix
* ` names(df) = c('A', 'B', 'C') ` set dataframe column names

### CSV
``` r
df = read.csv('data.csv', header=T, sep=',')
write.csv(df, file="data.csv", row.names=FALSE, col.names=T)
write.table(df, file="data.csv", row.names=FALSE, col.names=F, sep=',')
```

### Functions
``` r
f = function(x) {
    return(x+2)
}
apply(df, array_index, function) - apply(head(df)[, 4:dim(CO2)[2]], 1, function(x){x+2})
sapply(vector, function) - sapply(1:5, function(x){x+2})
lapply(list, function) - lapply(list(1:5), function(x){x+2})
```

### Linear Regression
``` r
n = 100
x = 1:n
y = x + rnorm(n)

model = lm(y~x)
coeffs = model$coefficients
# summary(model)

plot(x, y, type='l', col='blue', main='Example', xlab='x label', ylab='y label')
lines(x, coeffs[[1]] + coeffs[2]*x, col='green', type='l')
```

### Reshape
``` r
require(reshape2)
df = airquality
df$days = 1:dim(df)[1]
df_melt = melt(df[,c(1:4, 7)], id=c('days'), variable.name='climate_variable', value.name='climate_value')
df = dcast(df_melt, days ~ climate_variable)
```

### Plotting
**ggplot2**
``` r
x11(width=12, height=8)
require(ggplot2)
plot = ggplot(data=df_melt) +
       aes(x=days, y=climate_value) +
       geom_line(aes(colour=climate_variable)) +
       geom_smooth(method='lm', formula=y~x) +
       xlab('Days') +
       ylab('Climate Value') +
       scale_colour_manual(values=c('blue', 'orange', 'yellow', 'red'))

PLOT_NAME = 'practice.png'
png(filename=PLOT_NAME, width=1600, height=900)
rm(PLOT_NAME)
plot
dev.off()
```
**Plotly** - Interactive 3D Plot
``` r
require(plotly)

# Sample Data
n = 100
x = 1:n
y = 1:n
z = 0.01*rnorm(n^2) + 1
colors = rnorm(n^2) + 1

# Plot
plot_ly(x=x, y=y, z=matrix(z, ncol=n),
        surfacecolor=matrix(colors, ncol=n), colors=c('red', 'green'), cmin=0, cmax=2,
        colorbar=list(title='Intensity', tickmode='array', tickvals=seq(0, 2, by=0.25)), type='surface')
%>% layout(title = 'Surface Plot Example',
           scene = list(xaxis = list(title = 'x label'),
                        yaxis = list(title = 'y label'),
                        zaxis = list(title = 'random', range=c(0.9, 1.1)))
          )
```

### Aggregation
``` r
require(dplyr)
as.data.frame(df %>%
              select(col_name_1, col_name_2) %>%
              group_by(col_name_1) %>%
              summarise(col_name_2=round(mean(col_name_2, na.rm=T), 2))
)
```

### Database Connections
**RMySQL**
```r
require(RMySQL)

conn = dbConnect(MySQL(), user=USERNAME, password=PASSWORD, host='127.0.0.1', port=PORT)

profileQuery = 'SELECT * FROM intersect_filt'
profileData = fetch(dbSendQuery(conn, profileQuery), n=-1)
```
**RODBC**
```r
require(RODBC)

DSN = 'DATABASE_NAME'
USERNAME = 'USERNAME'
PASSWORD = 'PASSWORD'
conn = RODBC::odbcConnect(DSN, USERNAME, PASSWORD)
rm(DSN, USERNAME, PASSWORD)

query = 'SELECT * FROM TABLE'
results = RODBC::sqlQuery(conn, query)
rm(query)

RODBC::odbcClose(conn)
```
**RSQLite**
```r
require(RSQLite)

DATABASE_PATH = 'DATABASE_NAME.db'
db = RSQLite::dbConnect(RSQLite::dbDriver("SQLite"), DATABASE_PATH)
rm(DATABASE_PATH)

query = 'SELECT * FROM TABLE'
results = RSQLite::dbGetQuery(db, query)
rm(query)

RSQLite::dbDisconnect(db)
```
