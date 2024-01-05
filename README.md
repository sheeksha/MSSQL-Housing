# MSSQL-Housing

### DATA USED FROM : https://www.kaggle.com/datasets/tmthyjames/nashville-housing-data
### Nashville Housing Data ~ Home value data for the booming Nashville market
### Cleaning Data in SQL Queries

--------------------------------------------------------------------------------------------------------------------------

### Visualise the whole dataset
```
Select top 100* 
From HousingPriceProject.dbo.NashvilleHousing with (nolock)
```
--------------------------------------------------------------------------------------------------------------------------

### Standardize SaleDate Format

1. Visualise the SaleDate

```
Select SaleDate, CONVERT(Date,SaleDate) AS SaleDateOnly
From HousingPriceProject.dbo.NashvilleHousing
```

3. Update the SaleDate to Date-Only format (sometimes this command does not work)

```
Update NashvilleHousing
SET SaleDate = CONVERT(Date,SaleDate)
```

4. Alter the table to add a new column

```
ALTER TABLE NashvilleHousing
Add SaleDateConverted Date;
```

5. Convert the SaleDate into Date-Only and put it in the new column

```
Update NashvilleHousing
SET SaleDateConverted = CONVERT(Date,SaleDate)
```

7. Visualise the new column
```
Select SaleDateConverted
From HousingPriceProject.dbo.NashvilleHousing
```
 --------------------------------------------------------------------------------------------------------------------------

### Populate Property Address data

1. Data for which the PropertyAddress is NULL
```
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
Where PropertyAddress is NULL
```
3. Housing having unique ParcelID have the same PropertyAddress
```
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
order by ParcelID
```

4. Join the tables to check whether the NULL values can be found from another row
```
Select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing a 
Join HousingPriceProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	and a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is NULL
```

5. Update the table to populate the empty PropertyAddress
```
Update a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
From HousingPriceProject.dbo.NashvilleHousing a 
Join HousingPriceProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	and a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is NULL
```
--------------------------------------------------------------------------------------------------------------------------

### Breaking out Property Address into Individual Columns (Address, City)

1. The property address column
```
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
```

2. Selecting only the address from the property address.
- Selecting the characters starting at position 1 and ending right before the comma
```
Select
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address
From HousingPriceProject.dbo.NashvilleHousing
```

3. Selecting only the city from the property address
- Selecting the characters starting after the comma and ending the last character in in the PropertyAddress
```
Select
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) as City
From HousingPriceProject.dbo.NashvilleHousing
```

4. Alter the table to add two new columns
```
ALTER TABLE NashvilleHousing
Add PropertyAddressOnly Nvarchar(255), PropertyCityOnly Nvarchar(255);
```

5. Putting it in the new column
```
Update NashvilleHousing
SET PropertyAddressOnly = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

Update NashvilleHousing
SET PropertyCityOnly = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))
```

6. Visualise the new columns
```
Select *
From HousingPriceProject.dbo.NashvilleHousing
```

7. Breaking out Owner Address into Individual Columns (Address, City, State)
- We use a simpler method PARSENAME instead of SUBSTRING
```
Select OwnerAddress 
From HousingPriceProject.dbo.NashvilleHousing
```

8. Use PARSENAME to get characters between periods.
- Replace comma by period
```
Select
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)
,PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)
, PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
From HousingPriceProject.dbo.NashvilleHousing
```

9. Alter the table to add two new columns
```
ALTER TABLE NashvilleHousing
Add OwnerAddressOnly Nvarchar(255), OwnerCityOnly Nvarchar(255), OwnerStateOnly Nvarchar(255) ;
```

10. Putting it in the new column
```
Update NashvilleHousing
SET OwnerAddressOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

Update NashvilleHousing
SET OwnerCityOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

Update NashvilleHousing
SET OwnerStateOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
```

11. Visualise the new columns
```
Select *
From HousingPriceProject.dbo.NashvilleHousing
```
--------------------------------------------------------------------------------------------------------------------------

### Change Y and N to Yes and No in "Sold as Vacant" field

1. Check the distinct data in the SoldAsVacant column
```
Select Distinct(SoldAsVacant), Count(SoldAsVacant)
From HousingPriceProject.dbo.NashvilleHousing
Group By SoldAsVacant
Order By 2
```

2. If the data entered  as 'Y' convert it to 'Yes' and 'N' convert to 'No'
```
Select SoldAsVacant, Count(SoldAsVacant)
, CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
From HousingPriceProject.dbo.NashvilleHousing
Group By SoldAsVacant
Order By 2
```

3. Update the table
```
Update NashvilleHousing
SET SoldAsVacant = CASE When SoldAsVacant = 'Y' THEN 'Yes'
						When SoldAsVacant = 'N' THEN 'No'
						ELSE SoldAsVacant
						END
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------

### Remove Duplicates

1. Visualise the duplicates
```
Select *, 
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SaleDate,
				 SalePrice,
				 LegalReference
				 Order By UniqueID)
				 row_num
From HousingPriceProject.dbo.NashvilleHousing
Order By ParcelID, row_num
```

2. Create a CTE to ensure put all the duplicates in a temporary table for visualisation
```
WITH RowNumCTE AS(
Select *, 
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SaleDate,
				 SalePrice,
				 LegalReference
				 Order By UniqueID)
				 row_num
From HousingPriceProject.dbo.NashvilleHousing
)

Select *
From RowNumCTE
Where row_num > 1
```

3. Delete all duplicate
** Caution : To always backup raw data before deleting **
```
WITH RowNumCTE AS(
Select *, 
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SaleDate,
				 SalePrice,
				 LegalReference
				 Order By UniqueID)
				 row_num
From HousingPriceProject.dbo.NashvilleHousing
)
Delete
From RowNumCTE
Where row_num > 1
```

---------------------------------------------------------------------------------------------------------

### Delete Unused Columns

```
Select *
From HousingPriceProject.dbo.NashvilleHousing

Alter Table HousingPriceProject.dbo.NashvilleHousing
Drop Column PropertyAddress, SaleDate, OwnerAddress, TaxDistrict
```


















