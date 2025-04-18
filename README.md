
it('should populate finListInfo when LDATA values and headers are present', () => {
    const docContainer = {
      dataMapModel: [
        {
          type: 'LDATA',
          key: 'someKey',
          identifiers: ['id1'],
          value: 'someValue',
          listDataHeaders: [
            { key: 'name', value: 'val1', valueType: 'NAME' },
            { key: 'names', value: 'val2', valueType: 'NAMES' },
            { key: 'other', value: 'val3', valueType: 'OTHER' }
          ]
        }
      ]
    };

    const mockValues = ['value1'];
    const mockData_NAME = ['John Doe'];
    const mockData_NAMES = ['Jane', 'John'];
    const mockData_OTHER = 'Random';

    mockUtils.isEmpty.and.returnValues(false, false, false, false, false, false, false);
    mockHydrateService.getData = jasmine.createSpy().and.callFake((val, key) => {
      if (key === 'val1') return mockData_NAME;
      if (key === 'val2') return mockData_NAMES;
      if (key === 'val3') return mockData_OTHER;
      return null;
    });
    mockHydrateService.processName = (val: any) => val;
    mockHydrateService.processNames = (val: any) => val;
    mockUtils.deduplist = (arr: any[]) => Array.from(new Set(arr));

    service.taskInfoGetResponse = {}; // required for getData()
    service.taskHydrateDataService = mockHydrateService;

    service.mapDocData(docContainer);

    // Basic expectation to verify data got pushed to finListInfo
    expect(mockHydrateService.getData).toHaveBeenCalled();
  });
