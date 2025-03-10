describe('getData', () => {

  it('should return getNodes when applyFilter is false', () => {
    const data = { key: 'value' };
    const searchPath = 'key';
    const filterPath = undefined;
    const filterValue = undefined;

    spyOn(service, 'getNodes').and.returnValue('mockNodes');

    const result = service.getData(data, searchPath, filterPath, filterValue);

    expect(service.getNodes).toHaveBeenCalledWith(data, ['key']);
    expect(result).toBe('mockNodes');
  });

  it('should find common prefix and split search and filter paths correctly', () => {
    const searchPath = 'user.address.street';
    const filterPath = 'user.address.city';

    spyOn(service, 'findCommonPrefix').and.returnValue(2);

    const commonPath = service.findCommonPrefix(searchPath.split('.'), filterPath.split('.'));

    expect(commonPath).toBe(2);
  });

  it('should process nodes and filter values correctly', () => {
    const data = { users: [{ name: 'John' }, { name: 'Doe' }] };
    const searchPath = 'users.name';
    const filterPath = 'users.name';
    const filterValue = 'John';

    spyOn(service, 'getNodes').and.returnValue([{ name: 'John' }, { name: 'Doe' }]);
    spyOn(service, 'getValuesFromNode').and.callFake((node, subPath) => node[subPath]);

    const result = service.getData(data, searchPath, filterPath, filterValue);

    expect(service.getNodes).toHaveBeenCalled();
    expect(service.getValuesFromNode).toHaveBeenCalled();
    expect(result).toContain('John');
  });

  it('should process results when resFormat is true', () => {
    const data = { users: [{ name: 'John' }, { name: 'Doe' }] };
    const searchPath = 'users.name';
    const filterPath = undefined;
    const filterValue = undefined;
    const resFormat = true;

    spyOn(service, 'getNodes').and.returnValue([{ name: 'John' }, { name: 'Doe' }]);
    spyOn(service, 'processName').and.callFake((val) => val.toUpperCase());

    const result = service.getData(data, searchPath, filterPath, filterValue, resFormat);

    expect(service.getNodes).toHaveBeenCalled();
    expect(service.processName).toHaveBeenCalledWith('John');
    expect(service.processName).toHaveBeenCalledWith('Doe');
    expect(result).toContain('JOHN');
    expect(result).toContain('DOE');
  });

  it('should return names when resFormat is true and result is a string', () => {
    const data = {};
    const searchPath = 'name';
    const resFormat = true;

    spyOn(service, 'getNodes').and.returnValue(['John', 'Doe']);

    const result = service.getData(data, searchPath, undefined, undefined, resFormat);

    expect(result).toEqual(['John', 'Doe']);
  });

});
