require('dotenv').config();
const express = require('express');
const { createClient } = require('@supabase/supabase-js');
const path = require('path');

const app = express();
const port = 3000;

// 读取环境变量
const SUPABASE_URL = process.env.SUPABASE_URL;
const SUPABASE_ANON_KEY = process.env.SUPABASE_ANON_KEY;

const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// 中间件
app.use(express.json());
app.use(express.static(path.join(__dirname, 'public')));

// 获取所有任务
app.get('/api/tasks', async (req, res) => {
  const { data, error } = await supabase
    .from('tasks')
    .select('*')
    .order('created_at', { ascending: false });
  res.json({data, error});
});

// 添加任务
app.post('/api/tasks', async (req, res) => {
  const { task_title, task_desc } = req.body;
  const { data, error } = await supabase
    .from('tasks')
    .insert([{ task_title, task_desc }])
    .select();
  res.json({data, error});
});

// 删除任务
app.delete('/api/tasks/:id', async (req, res) => {
  const { id } = req.params;
  const { error } = await supabase
    .from('tasks')
    .delete()
    .eq('id', id);
  res.json({error});
});

// 修改任务完成状态
app.patch('/api/tasks/:id', async (req, res) => {
  const { id } = req.params;
  const { is_completed } = req.body;
  const { data, error } = await supabase
    .from('tasks')
    .update({ is_completed })
    .eq('id', id)
    .select();
  res.json({data, error});
});

app.listen(port, () => {
  console.log(`服务启动，访问 http://localhost:${port}`);
});
