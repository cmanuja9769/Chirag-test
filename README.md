 {
    {!isError && return (    
      <>
        <Box component="main" sx={{ flexGrow: 1, display: 'flex', justifyContent: 'center', alignItems: 'center', width: '70vw' }}>
          <DataTable
            columns={data?.columns}
            rowsData={filteredRows?.length ? filteredRows : data?.rowsData ? data?.rowsData : []}
            handleSearchInputChange={handleSearchInpChange}
            searchValue={searchvalue}
            handleDialog={handleDialog}
            title={t('dagTitle')}
            showCreateBtn={true}
            createBtnLabel={t('newDag')}
            showSearch={true}
            showPagination={true}
            pageSize={10}
          />
          <DialogComponent showDialog={showDialog} handleClick={handleDialogClick} title={t('confirmation')}>
            {t('confirmMessage')}
          </DialogComponent>
        </Box>
      </>
    ) :
    handleToast("error", TOAST_TYPE.ERROR) return (
      <>
        <Box component="main" sx={{ flexGrow: 1, display: 'flex', justifyContent: 'center', alignItems: 'center', width: '70vw' }}>
          <DataTable
            columns={[]}
            rowsData={[]}
            handleSearchInputChange={handleSearchInpChange}
            searchValue={searchvalue}
            handleDialog={handleDialog}
            title={t('dagTitle')}
            showCreateBtn={true}
            createBtnLabel={t('newDag')}
            showSearch={true}
            showPagination={true}
            pageSize={10}
          />
        </Box>
        <ToastMessage severity={toastInfo.severity} isVisible={openToast} hideToast={() => setOpenToast(false)} message={toastInfo.message} />
      </>
    );}
  }
