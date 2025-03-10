describe('hydrateData', () => {
  
  it('should call getData for VALIDATEANDSET', () => {
    const dataMapModel = {
      type: 'VALIDATEANDSET',
      value: 'someValue',
      key: 'someKey',
      identifiers: 'someIdentifiers'
    };
    
    spyOn(service, 'getData').and.callThrough();
    service.hydrateData(dataMapModel, aggregatedTaskData);
    
    expect(service.getData).toHaveBeenCalledWith(
      aggregatedTaskData, 
      dataMapModel.value, 
      dataMapModel.key, 
      dataMapModel.identifiers, 
      true
    );
  });

  it('should return "Yes" for BOOLEAN when valueObj is true', () => {
    const dataMapModel = {
      type: 'BOOLEAN',
      value: true
    };

    const result = service.hydrateData(dataMapModel, aggregatedTaskData);
    expect(result).toBe('Yes');
  });

  it('should return "No" for BOOLEAN when valueObj is false', () => {
    const dataMapModel = {
      type: 'BOOLEAN',
      value: false
    };

    const result = service.hydrateData(dataMapModel, aggregatedTaskData);
    expect(result).toBe('No');
  });

  it('should prepend "$" for CURRENCY type', () => {
    const dataMapModel = {
      type: 'CURRENCY',
      value: 100
    };

    const result = service.hydrateData(dataMapModel, aggregatedTaskData);
    expect(result).toBe('$100');
  });

});
