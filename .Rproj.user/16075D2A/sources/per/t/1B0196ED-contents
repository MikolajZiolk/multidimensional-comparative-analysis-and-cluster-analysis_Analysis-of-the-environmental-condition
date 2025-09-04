library(outliers)
library(caret) #do korelacji
library(factoextra) #do wykresu klastroewgo
library(cluster) # do PAM
library(psych)
library(corrplot) #do korelacji


data <- read.csv("C:/Users/mikol/OneDrive/Desktop/ProjektSad/DaneCSV1.csv", sep = ";")
head(data)
View(data)

#tylko klumny z wartości numeryczne
data_numeric <- data[, c("X1", "X2", "X3", "X4", "X5", "X6", "X7", "X8", "X9")]
str(data_numeric)

clean_numeric_column <- function(col) {
  col <- gsub(",", ".", col)  # Zamiana przecinków na kropki
  col <- gsub(" ", "", col)   # Usunięcie spacji
  return(as.numeric(col))     # Konwersja na numeric
}

# Zastosowanie funkcji do kolumn, które są zapisane jako 'character'
data_numeric$X1 <- clean_numeric_column(data_numeric$X1)
data_numeric$X2 <- clean_numeric_column(data_numeric$X2)
data_numeric$X3 <- clean_numeric_column(data_numeric$X3)
data_numeric$X5 <- clean_numeric_column(data_numeric$X5)
data_numeric$X6 <- clean_numeric_column(data_numeric$X6)
data_numeric$X8 <- clean_numeric_column(data_numeric$X8)
data_numeric$X9 <- clean_numeric_column(data_numeric$X9)

str(data_numeric)

describe(data_numeric)
data_numeric |> summary() 


cor_matrix <- cor(data_numeric)
cor_matrix

# Wizualizacja macierzy korelacji (mapa cieplna)

corrplot(cor_matrix, method = "color", type = "upper", 
         tl.col = "black", tl.srt = 45, addCoef.col = "black")  # Dodanie wartości współczynników korelacji


#cv-współczynnik zmienności
cv <- function(x) {
  return ((sd(x) / mean(x)) * 100)
}

#cv dla każdej kolumny
cv_values <- apply(data_numeric, 2, cv)
print(cv_values) #wszystkie wartosci wieksze od 10

# Boxploty dla każdej zmiennej (dodatkowo ,plus sa również w excelu)
par(mfrow = c(3, 3))
for (i in 1:ncol(data_numeric)) {
  boxplot(data_numeric[, i], main = colnames(data_numeric)[i], ylab = "Wartość", col = "lightblue")
}

# Test Grubbsa dla każdej zmiennej ->wykrywanie wartości odstających
library(outliers)
for (i in 1:ncol(data_numeric)) {
  test <- grubbs.test(data_numeric[, i])
  print(paste("Grubbs test for", colnames(data_numeric)[i], ": p-value =", test$p.value))
}

# Jeśli p-value < 0.05, to sugeruje istnienie wartości odstających


##############################
#zamiana wartości odstających
# Funkcja do zamiany wartości odstających na wartości końca wąsa
replace_outliers <- function(x) {
  Q1 <- quantile(x, 0.25) 
  Q3 <- quantile(x, 0.75)  
  IQR <- Q3 - Q1           
  
  lower_whisker <- Q1 - 1.5 * IQR  
  upper_whisker <- Q3 + 1.5 * IQR  
  
  x[x < lower_whisker] <- lower_whisker
  
  x[x > upper_whisker] <- upper_whisker
  
  return(x)
}

# Zastosowanie funkcji do każdej kolumny w data_numeric
data_no_outliers <- as.data.frame(lapply(data_numeric, replace_outliers))

# Sprawdzenie zmiennych po zamianie wartości odstających
summary(data_no_outliers)
describe(data_no_outliers)
for (i in 1:ncol(data_no_outliers)) {
  test <- grubbs.test(data_no_outliers[, i])
  print(paste("Grubbs test for", colnames(data_no_outliers)[i], ": p-value =", test$p.value))
}

par(mfrow = c(3, 3))
for (i in 1:ncol(data_no_outliers)) {
  boxplot(data_no_outliers[, i], main = colnames(data_no_outliers)[i], ylab = "Wartość", col = "lightblue")
}

##########################################
cor(data_no_outliers)

data_no_outliers1 <- read_excel("C:/Users/mikol/OneDrive/Desktop/ProjektSad/data_no_outliers.xlsx")

#
# Wyświetlanie wyników
print(wyniki)

# Standaryzacja danych
data_scaled <- scale(data_no_outliers)

# Elbow Method
fviz_nbclust(data_scaled, kmeans, method = "wss") +
  labs(subtitle = "Elbow Method")

# Silhouette Method
fviz_nbclust(data_scaled, kmeans, method = "silhouette") +
  labs(subtitle = "Silhouette Method")

#k-means
set.seed(123)  
kmeans_result <- kmeans(data_scaled, centers = 4, nstart = 5)
print(kmeans_result)
data$cluster_kmeans <- kmeans_result$cluster

data$cluster_kmeans

# Wyświetlenie powiatów przypisanych do poszczególnych klastrów dla k-means
for (i in 1:4) {  # tutaj jest 4, bo centers = 4
  cat(paste("Klastr", i, "dla k-means:\n"))
  print(data$Powiaty[data$cluster_kmeans == i])
  cat("\n")
}

#algorytm k-medoid (PAM)
set.seed(123)  
pam_result <- pam(data_scaled, k = 4)  

print(pam_result)

data$cluster <- pam_result$clustering

fviz_cluster(pam_result, data = data_scaled)

pam_result$clustering
pam_result$medoids
pam_result$data
pam_result$clusinfo


#algorytm hierarchiczny Warda

dist_euc <- dist(data_scaled, method="euclidean") #odległość danych

hier_clust_result <- hclust(dist_euc, method="ward.D2") #metoda warda



plot(hier_clust_result,
     main = "Dendrogram - Hierarchiczne grupowanie metodą Warda", 
     xlab = "Powiaty",
     ylab = "Odległość",
     cex.lab = 1,
     las = 2,
     cex.axis = 0.9,
     labels = data$Powiaty)

abline(h = 5, col = "red", lwd = 2)# linia do wykresu

clusters<-cutree(hier_clust_result, k = 7)


# Dodanie informacji o klastrach do danych (możliwe przypisanie klastra do każdej obserwacji)
data_no_outliers$Cluster <- clusters

# Wyświetlenie powiatów przypisanych do poszczególnych klastrów
for (i in 1:7) {
  cat(paste("Klastr", i, ":\n"))
  print(data$Powiaty[clusters == i])
  cat("\n")
}


# Zakładamy, że zmienne X1-X9 są w data_no_outliers, a kolumna 'cluster' wskazuje do którego klastra należy powiat
library(dplyr)

# Obliczanie średnich i odchylenia standardowego dla zmiennych X1-X9 w każdym klastrze
statystyki_klastry <- data_no_outliers %>%
  group_by(Cluster) %>%
  summarise(
    mean_X1 = mean(X1, na.rm = TRUE),
    mean_X2 = mean(X2, na.rm = TRUE),
    mean_X3 = mean(X3, na.rm = TRUE),
    mean_X4 = mean(X4, na.rm = TRUE),
    mean_X5 = mean(X5, na.rm = TRUE),
    mean_X6 = mean(X6, na.rm = TRUE),
    mean_X7 = mean(X7, na.rm = TRUE),
    mean_X8 = mean(X8, na.rm = TRUE),
    mean_X9 = mean(X9, na.rm = TRUE),
    sd_X1 = sd(X1, na.rm = TRUE),
    sd_X2 = sd(X2, na.rm = TRUE),
    sd_X3 = sd(X3, na.rm = TRUE),
    sd_X4 = sd(X4, na.rm = TRUE),
    sd_X5 = sd(X5, na.rm = TRUE),
    sd_X6 = sd(X6, na.rm = TRUE),
    sd_X7 = sd(X7, na.rm = TRUE),
    sd_X8 = sd(X8, na.rm = TRUE),
    sd_X9 = sd(X9, na.rm = TRUE)
  )

# Wyświetlanie wyników
print(statystyki_klastry)

