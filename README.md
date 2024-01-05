# MSSQL-Housing

### DATA USED FROM : https://www.kaggle.com/datasets/tmthyjames/nashville-housing-data
### Nashville Housing Data ~ Home value data for the booming Nashville market
### Cleaning Data in SQL Queries

--------------------------------------------------------------------------------------------------------------------------

### Visualise the whole dataset
```ruby
Select top 100* 
From HousingPriceProject.dbo.NashvilleHousing with (nolock)
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/f672f530-18fa-400d-95fe-2ef135934664)

--------------------------------------------------------------------------------------------------------------------------

### Standardize SaleDate Format

1. Visualise the SaleDate

```ruby
Select SaleDate, CONVERT(Date,SaleDate) AS SaleDateOnly
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/718ee424-f2d4-49bb-8b92-9ba17e98d3bf)

3. Update the SaleDate to Date-Only format (sometimes this command does not work)

```ruby
Update NashvilleHousing
SET SaleDate = CONVERT(Date,SaleDate)
```

4. Alter the table to add a new column

```ruby
ALTER TABLE NashvilleHousing
Add SaleDateConverted Date;
```

5. Convert the SaleDate into Date-Only and put it in the new column

```ruby
Update NashvilleHousing
SET SaleDateConverted = CONVERT(Date,SaleDate)
```

7. Visualise the new column
```ruby
Select SaleDateConverted
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/25fa5502-a9ce-4a1c-af21-c46e799274fc)

--------------------------------------------------------------------------------------------------------------------------

### Populate Property Address data

1. Data for which the PropertyAddress is NULL
```ruby
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
Where PropertyAddress is NULL
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/05d8c24e-2b3e-4a40-978d-5d6c4ecb9f8d)

3. Housing having unique ParcelID have the same PropertyAddress
```ruby
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
order by ParcelID
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/25a6449c-d3fc-4b04-a7b0-9fdf08081344)

4. Join the tables to check whether the NULL values can be found from another row
```ruby
Select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing a 
Join HousingPriceProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	and a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is NULL
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/786dd4d4-a382-4c51-a0bd-f3d2c9495457)

5. Update the table to populate the empty PropertyAddress
```ruby
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
```ruby
Select PropertyAddress
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/e86a9422-e64c-4c81-aacc-0b99f3c951e4)

2. Selecting only the address from the property address.
- Selecting the characters starting at position 1 and ending right before the comma
```ruby
Select
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/79b17ff1-70fd-43d2-9a8d-bc035e475106)

3. Selecting only the city from the property address
- Selecting the characters starting after the comma and ending the last character in in the PropertyAddress
```ruby
Select
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) as City
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/82fe369a-7b7e-461e-9fc5-7144aff43953)

4. Alter the table to add two new columns
```ruby
ALTER TABLE NashvilleHousing
Add PropertyAddressOnly Nvarchar(255), PropertyCityOnly Nvarchar(255);
```

5. Putting it in the new column
```ruby
Update NashvilleHousing
SET PropertyAddressOnly = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

Update NashvilleHousing
SET PropertyCityOnly = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))
```

6. Visualise the new columns
```ruby
Select *
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/a2a8d0e7-3938-47c5-9b01-1434fe807afb)

7. Breaking out Owner Address into Individual Columns (Address, City, State)
- We use a simpler method PARSENAME instead of SUBSTRING
```ruby
Select OwnerAddress 
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/e46f50ea-092c-4306-a8b8-a45d83023fae)

8. Use PARSENAME to get characters between periods.
- Replace comma by period
```ruby
Select
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)
,PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)
, PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/b223a407-5b04-4129-9f96-fcde8438bc55)

9. Alter the table to add two new columns
```ruby
ALTER TABLE NashvilleHousing
Add OwnerAddressOnly Nvarchar(255), OwnerCityOnly Nvarchar(255), OwnerStateOnly Nvarchar(255) ;
```

10. Putting it in the new column
```ruby
Update NashvilleHousing
SET OwnerAddressOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

Update NashvilleHousing
SET OwnerCityOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

Update NashvilleHousing
SET OwnerStateOnly = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
```

11. Visualise the new columns
```ruby
Select *
From HousingPriceProject.dbo.NashvilleHousing
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/91b654cf-8cd4-405b-ade8-83e9bdf06819)

--------------------------------------------------------------------------------------------------------------------------

### Change Y and N to Yes and No in "Sold as Vacant" field

1. Check the distinct data in the SoldAsVacant column
```ruby
Select Distinct(SoldAsVacant), Count(SoldAsVacant)
From HousingPriceProject.dbo.NashvilleHousing
Group By SoldAsVacant
Order By 2
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/014387fa-be90-4ef1-9907-ff334426ecd6)

2. If the data entered  as 'Y' convert it to 'Yes' and 'N' convert to 'No'
```ruby
Select SoldAsVacant, Count(SoldAsVacant)
, CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
From HousingPriceProject.dbo.NashvilleHousing
Group By SoldAsVacant
Order By 2
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/530074ea-9cb4-4805-a1ab-5a9d924b3f63)

3. Update the table
```ruby
Update NashvilleHousing
SET SoldAsVacant = CASE When SoldAsVacant = 'Y' THEN 'Yes'
						When SoldAsVacant = 'N' THEN 'No'
						ELSE SoldAsVacant
						END
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/4ee951e6-f6b2-4f48-86e1-fa202aad03ab)

-----------------------------------------------------------------------------------------------------------------------------------------------------------

### Remove Duplicates

1. Visualise the duplicates
```ruby
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
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/db0210f4-b1d6-4679-a902-a819714367bf)

2. Create a CTE to ensure put all the duplicates in a temporary table for visualisation
```ruby
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
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/b76f9cbf-78f9-4896-94a5-04c7e0443bf9)

3. Delete all duplicate
** Caution : To always backup raw data before deleting **
```ruy
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

```ruby
Select *
From HousingPriceProject.dbo.NashvilleHousing

Alter Table HousingPriceProject.dbo.NashvilleHousing
Drop Column PropertyAddress, SaleDate, OwnerAddress, TaxDistrict
```
![image](https://github.com/sheeksha/MSSQL-Housing/assets/69764380/80ff0cdc-1618-41a4-a6e3-feca63cf071b)



















