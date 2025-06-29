<script>
  import { usePagination } from 'alova/client';
  import { writable } from 'svelte/store';
  import { queryStudents, removeStudent } from '../../api/methods';
  import Table from '../../components/Table.svelte';
  import EditorModal from './EditorModal.svelte';

  const studentName = writable('');
  const clsName = writable('');
  let detailVisible = false;
  let selectedId;

  const {
    loading,
    data: students,
    page,
    total,
    pageSize,
    pageCount,
    remove,
    insert,
    refresh,
    reload,
    removing
  } = usePagination((page, pageSize) => queryStudents(page, pageSize, $studentName, $clsName), {
    watchingStates: [studentName, clsName],
    initialData: { total: 0, list: [] },
    debounce: [800],
    total: res => res.total,
    data: res => res.list,
    actions: {
      remove: ({ id }) => removeStudent(id)
    }
  });

  const editItem = id => {
    detailVisible = true;
    selectedId = id;
  };

  const updateList = ({ detail }) => {
    if (selectedId) {
      refresh($page);
    } else {
      insert(detail);
    }
    detailVisible = false;
  };

  const onPageChange = (newPage, newPageSize) => {
    $page = newPage;
    $pageSize = newPageSize;
  };

  const classOptions = [
    {
      title: 'class 1',
      value: 'class1'
    },
    {
      title: 'class 2',
      value: 'class2'
    },
    {
      title: 'class 3',
      value: 'class3'
    }
  ];
  const columns = [
    {
      title: 'ID',
      dataIndex: 'id'
    },
    {
      title: 'Name',
      dataIndex: 'name'
    },
    {
      title: 'Class',
      key: 'cls',
      dataIndex: 'cls'
    },
    {
      title: 'Operate',
      dataIndex: 'operate',
      render: 'slot4'
    }
  ];
</script>

<div>
  <div class="grid gap-y-4">
    <div class="grid grid-cols-[repeat(5,fit-content(100px))] gap-x-2">
      <nord-input
        hide-label
        value={$studentName}
        on:input={({ target }) => ($studentName = target.value)}
        placeholder="input student name" />
      <nord-select
        placeholder="select class"
        value={$clsName}
        on:change={({ target }) => ($clsName = target.value)}
        hide-label>
        {#each classOptions as option}
          <option value={option.value}>
            {option.title}
          </option>
        {/each}
      </nord-select>
      <nord-button
        variant="primary"
        on:click={() => editItem(null)}>Add Item</nord-button>
      <nord-button
        variant="primary"
        on:click={reload}>Reload</nord-button>
    </div>

    <Table
      loading={$loading}
      {columns}
      data={$students}
      pagination={{
        page: $page,
        pageSize: $pageSize,
        onChange: onPageChange,
        pageCount: $pageCount,
        total: $total,
        pageSizes: [10, 20]
      }}>
      <div
        class="grid grid-cols-[repeat(2,fit-content(100px))] gap-x-2"
        slot="slot4"
        let:row
        let:index>
        <nord-button
          size="s"
          on:click={() => editItem(row.id)}>
          Edit
        </nord-button>
        <nord-button
          variant="danger"
          size="s"
          type="error"
          disabled={$removing.includes(index) || undefined}
          on:click={() => remove(row)}>
          Remove
        </nord-button>
      </div>
    </Table>
  </div>

  <nord-modal
    open={detailVisible || undefined}
    on:close={() => (detailVisible = false)}>
    <h3 slot="header">Student Info</h3>
    {#if detailVisible}
      <EditorModal
        id={selectedId}
        on:submit={updateList} />
    {/if}
  </nord-modal>
</div>
